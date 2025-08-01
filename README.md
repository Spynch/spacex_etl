# SpaceX ETL

Этот репозиторий содержит пример ETL-пайплайна для загрузки данных из публичного API SpaceX в аналитическое хранилище. В инфраструктуре используются Docker Compose, Apache Airflow, PostgreSQL, ClickHouse, dbt и Metabase.

## Структура проекта

- **airflow/** – DAG-файлы для Airflow. `dags_db_spacex_api.py` загружает данные из API SpaceX в базу PostgreSQL, `dags_create_data_marts.py` запускает dbt для построения витрин.
- **postgres/** – скрипты инициализации двух серверов PostgreSQL: `server_publicist` (источник данных) и `server_subscription` (подписчик логической репликации).
- **clickhouse/** – скрипт подключения таблиц ClickHouse к PostgreSQL через движок `PostgreSQL`.
- **dbt/** – проект dbt с моделями и макросами для формирования витрин данных в ClickHouse.
- **metabase/** – Dockerfile для Metabase c подключенным драйвером ClickHouse.
- **docker-compose.yaml** – описание всех сервисов (Airflow, PostgreSQL, ClickHouse, Metabase и др.) и сетей между ними.
- **Dockerfile** – образ Airflow с установленными зависимостями.
- **requirements.txt** – python-зависимости (dbt, requests и т.д.).

## Используемые технологии

- **Python** и **Apache Airflow** – управление расписанием задач ETL.
- **PostgreSQL** – хранение сырых данных и организация логической репликации.
- **ClickHouse** – аналитическое хранилище, подключенное к PostgreSQL.
- **dbt** – создание витрин данных (views) в ClickHouse.
- **Metabase** – BI-система для визуализации полученных данных.
- **Docker Compose** – оркестрация всех контейнеров проекта.

## Процесс выполнения

1. **Запуск контейнеров** – `docker-compose up` поднимает PostgreSQL, ClickHouse, Airflow, Metabase и вспомогательные сервисы.
2. **Загрузка данных** – DAG `dags_db_spacex_api.py` обращается к API SpaceX, преобразует полученный JSON и сохраняет данные в таблицы PostgreSQL `server_publicist`.
3. **Логическая репликация** – данные из `server_publicist` автоматически реплицируются в `server_subscription`, откуда их читает ClickHouse.
4. **Подключение ClickHouse** – скрипт `connect_table_psql.sh` создаёт таблицы ClickHouse с движком `PostgreSQL` и указывает на базу `server_subscription`.
5. **Построение витрин** – DAG `create_data_marts` выполняет `dbt run`, создавая представления и отчётные таблицы в ClickHouse (см. файлы в `dbt/spacex/models`).
6. **Анализ данных** – Metabase подключается к ClickHouse и позволяет строить дашборды на основе сформированных витрин.
