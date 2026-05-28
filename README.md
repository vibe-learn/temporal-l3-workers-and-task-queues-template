        # temporal — Worker и Task Queue

        Homework-шаблон для урока **l3_workers_and_task_queues** (Worker и Task Queue) на платформе Vibe Learn.

        ## Что делать

        На go.temporal.io/sdk собери воркер: worker.New(client, "order-tq", ...), зарегистрируй
OrderWorkflow и активности, запусти w.Run. Добавь вторую Task Queue "special-tq" и второй
воркер на ней; одну активность направь через ActivityOptions{TaskQueue: "special-tq"} —
покажи task routing. В README — как запустить несколько воркер-процессов на одной очереди
для масштабирования. Тесты на TestWorkflowEnvironment проверят: workflow исполняется при
зарегистрированных функциях; активность с заданной TaskQueue маршрутизируется на нужный пул
(через RegisterActivity на разных воркер-окружениях / OnActivity).

## Контекст (из transfer-задачи урока)

У тебя ML-сервис на Temporal. Workflow PreprocessAndInferWorkflow: (1) скачать датасет,
(2) препроцессинг на CPU (лёгкий), (3) инференс на GPU (тяжёлый, нужен GPU-бокс), (4) сохранить
результат. GPU-боксов мало и они дорогие; обычных CPU-воркеров — много. Сейчас всё крутится
на одной Task Queue, и GPU-боксы простаивают на скачивании/препроцессинге, пока CPU-задачи
отъедают их слоты.

**Вопрос:** перепроектируй маршрутизацию. Опиши:
(a) сколько Task Queue завести и какие воркеры что поллят;
(b) как направить именно активность инференса на GPU-бокс, оставив остальное на CPU-пуле;
(c) что произойдёт с workflow, если GPU-воркер упадёт во время инференса, и почему ничего
    не потеряется.

## Recap из урока

- **Worker** — твой процесс: long-poll'ит Task Queue, исполняет зарегистрированные workflow/activity, рапортует серверу. Сам Temporal Server код не запускает.
- **Task Queue — это просто имя**, именованный канал маршрутизации, а НЕ брокер сообщений: нет партиций, retention, ребаланса.
- Воркер **stateless** — всё durable-состояние в Temporal Server. Масштабирование = больше воркеров, поллящих ТУ ЖЕ очередь.
- По очереди ходят **workflow tasks** (продвинь оркестрацию) и **activity tasks** (сделай реальную работу); воркер берёт оба раздельными пулами.
- **Sticky execution** — кэш состояния workflow на воркере, чтобы не реплеить всю историю каждый раз; при потере кэша другой воркер делает полный replay. **Отдельная Task Queue** пинит активность к конкретному воркеру (GPU-бокс, хост с файлом).

        ## Как работать

        1. Платформа Vibe Learn создаёт копию этого репо в твоём GitHub-аккаунте по клику «Начать домашку» на странице урока (через GitHub `/generate`, codecrafters-pattern).
        2. Склонируй копию локально, реализуй TODO в `main.go` (workflow + активности), прогони тесты, запушь.
        3. CI (`.github/workflows/ci.yml`) запускает `go vet` + `go test ./...` на каждый push. Платформа слушает результат через webhook от GitHub Actions и обновляет статус домашки на странице урока.

        ## Локальное окружение

        - Go 1.22+
        - SDK: `go.temporal.io/sdk`
        - Docker + docker-compose — `docker compose up` поднимает Temporal dev server на `:7233` + Web UI на `:8233`. Адрес переопределяется через env `TEMPORAL_ADDRESS` (дефолт `localhost:7233`).
        - Юнит-тесты на `testsuite.TestWorkflowEnvironment` (активности замоканы) бегут в CI БЕЗ сервера; интеграционный тест включается через `TEMPORAL_INTEGRATION=1`.

        ## Запуск

        ```bash
        # Поднять локальный Temporal dev server + UI
        docker compose up -d
        # Web UI: http://localhost:8233

        # Прогнать тесты (юнит на TestWorkflowEnvironment — без сервера;
        # интеграционный включается через TEMPORAL_INTEGRATION=1)
        go test ./...
        TEMPORAL_INTEGRATION=1 go test ./...

        # Запустить воркер (регистрирует workflow + активности, слушает task queue)
        go run .
        ```

        ## Заметка автора

        Это baseline-шаблон, сгенерированный платформой. Бизнес-сущность задачи (что конкретно реализовать в `main.go`, какие тесты сделать строгими) расширяется по ходу итераций — параллельно с углублением теории урока.
