# y4et-soundclouda
# Учёт прослушиваний SoundCloud-реперов 

# *Cтрахов Роман 25-ивт-2-1*

## [Use-case диарамма](https://github.com/rom4ikahaha/y4et-soundclouda/blob/main/docs/digrams/diagusecase.png)

## [Блок-схема сбора метрик](https://github.com/rom4ikahaha/y4et-soundclouda/blob/main/docs/digrams/c6op_Metrik.png)

## [Блок-схема процесса “Отчет ‘Топ N-треков’”](https://github.com/rom4ikahaha/y4et-soundclouda/blob/main/docs/digrams/ot4et_top_n_trekov.png)

## [Таблица]()

__В этой лабораторной работе проектируется ИС, которая собирает и анализирует метрики прослушиваний треков рэп артистов на SoundCloud, чтобы быстро видеть динамику, сравнивать релизы и готовить отчёты.__

## **Предметная область**: 
- Показать на практическом примере, как спроектировать ИС: определить предметную область, роли и функции, а затем описать процессы в виде Use Case и блок схем, плюс наметить структуру БД и архитектуру.

## **Цели системы**:

- Централизованно собирать и хранить метрики по трекам и артистам.

- Быстро строить отчёты “топ N”, видеть тренды и пики, сравнивать релизы.

- Экспортировать результаты в CSV/XLSX/PNG и присылать уведомления о всплесках

## **Пользователи и роли**:

- Администратор: ведёт справочники артистов и треков, настраивает расписание сбора, управляет пользователями/ролями, словарями жанров/тегов.

- Аналитик/Менеджер: смотрит дашборды, формирует отчёты, сравнивает треки/артистов, настраивает фильтры и экспорт.

## **Основные функции**:

- Регистрация артистов и треков с валидацией ссылок SoundCloud.

- Плановый сбор метрик по расписанию, хранение фактов и пересчёт суточных агрегатов.

- Отчёты: топ N, сравнения, приросты, коэффициент вовлечённости, фильтры по периодам/жанрам/артистам, экспорт CSV/XLSX/PNG.

- Уведомления о всплесках и аномалиях.

## **Требования и ограничения**: 

- уникальность ссылок треков; целостность связей; индексы по времени и идентификаторам; в рамках лабораторной допускается работа на тестовых данных без реальных интеграций, с имитацией источника.

## **Архитектура**:

- Базовая схема: трёхслойная архитектура с отдельными фоновыми задачами (ETL)

- **Presentation (Web UI)** : интерфейс аналитика для дашбордов, топ N, сравнений, экспорта.

- **Business Logic (API)** : агрегирующие запросы, расчёт метрик, валидация, RBAC.

- **Data/Repository** : реляционная БД для справочников, фактов и суточных агрегатов.

- **ETL по расписанию** : инкрементальная загрузка метрик, нормализация, пересчёт агрегатов, аудит JobRun

## **Компоненты системы**: 

- Веб клиент аналитика: страницы “Дашборд”, “Топ N”, “Сравнение”, “Отчёты”, формы экспорта CSV/XLSX/PNG, фильтры по периоду/артистам/жанрам; авторизация по ролям.

- Backend API: REST/GraphQL эндпоинты для агрегатов и сравнений, сервис отчётов, модуль уведомлений, эндпоинты статусов ETL и ручного перезапуска задач (для демонстрации).

- ETL сервис: периодический сбор метрик (имитация или реальный HTTP клиент), запись в MetricFact, пересчёт DailyAggregate, логирование в JobRun, обработка ретраев и throttling.

- Хранилище данных: PostgreSQL со схемой Artist, Track, MetricFact, DailyAggregate, JobRun, User; уникальности и индексы по времени/идентификаторам.

## **Схема данных**:

| <!-- -->      | <!-- -->        | <!-- -->      | <!-- -->      | <!-- -->      | <!-- -->      | <!-- -->      | <!-- -->      |  <!-- -->      | 
|:------------- |:---------------:| :-------------:| :-------------:|  :-------------:| :-------------:| :-------------:| :-------------:| :-------------:|
| Artist         | id        | name      |  url  |  genre |  tags |  created | 
| Track         | id         | artistId        | title  | url |  release_date  | genre |  tags | active |
| Metric         | id        | trackId        |  ts  |  listens |  likes | reposts | comments | source |
| DailyAggregate  | id | trackId       |  date  |  listens_sum | likes_sum  | reposts_sum | comments_sum | engagement_rate |
| JobRun         | id     | job_name       | started_at  |  finished_at | status | error_num | 
| User         | id       | login       | role  | email | active |

Artist(id, name, url, genre, tags, created).​

Track(id, artistId, title, url, release_date, genre, tags, active).​

Metric(id, trackId, ts, listens, likes, reposts, comments, source).​

DailyAggregate(id, trackId, date, listens_sum, likes_sum, reposts_sum, comments_sum, engagement_rate).​

JobRun(id, job_name, started_at, finished_at, status, error_num).​

User(id, login, role, email, active).

## **Технологии и иструменты**:

- **Backend**: Python/FastAPI (pydantic, uvicorn) или Node.js/NestJS (TypeScript, class validator).

- **Frontend**: React/Vite + Chart.js/ECharts либо Vue/Vite + ApexCharts; таблицы — AG Grid/Material.

- **БД**: PostgreSQL; миграции — Alembic (Python) или Prisma/TypeORM (Node).

- **ETL**: cron/systemd timer для учебного прототипа; для расширения — Apache Airflow (docker compose).

- **Экспорт**: CSV/XLSX/PNG; логирование — структурированные JSON логи, хранение JobRun в БД.

## **Примеры API**

- *GET /dashboard?artistId&from&to — агрегаты для дашборда.*

- *GET /top?metric=listens|er&n=&from=&to= — топ N треков.*

- *GET /compare?entity=track|artist&ids=&metric=&from=&to= — сравнение.*

- *POST /alerts — настройки уведомлений; GET /etl/status, POST /etl/run — сервисные операции.*

## **План реализации**

- **Этап 1:** Справочники Artist/Track, CRUD, валидация URL; БД и миграции.

- **Этап 2:** Таблицы MetricFact/DailyAggregate, индексы и уникальные ключи; генерация тестовых данных/имитация загрузки.

- **Этап 3:** ETL скрипт по расписанию, ретраи, throttling, журнал JobRun.

- **Этап 4:** Дашборды, отчёты Top N и сравнения; экспорт CSV/XLSX/PNG.

- **Этап 5:** Уведомления (пороговые правила), RBAC (Admin/Analyst), подготовка презентации




