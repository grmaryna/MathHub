## Побудова sequence-діаграми на REQ-32-history

### Промпт

Побудуй Mermaid sequenceDiagram на сценарій REQ-32 за [spec](spec.md)

### Вихід

Діаграму для REQ-32-history я зробила: файл diagram-req-32-history.mmd. Рендер я не перевіряла, тож встав код у Mermaid Live Editor і подивись, чи все відображається.

Вона побудована за таблицею кроків зі spec.md, у тому ж стилі, що й твої діаграми REQ-31 і REQ-32:

sequenceDiagram
    autonumber
    actor Student
    participant Scheduler
    participant SubscriptionService
    participant Subscription
    participant Homework_submission

    Note over Scheduler,Subscription: Фаза 1: щоденна перевірка підписок (від end_date минуло 8 діб)

    Scheduler->>+SubscriptionService: runDailyCheck()
    SubscriptionService->>+Subscription: findExpired(today)
    Subscription-->>-SubscriptionService: список прострочених підписок (статус active або grace)

    loop для кожної простроченої підписки
        SubscriptionService->>+Subscription: findLatestSubscription(studentId, courseId)
        Subscription-->>-SubscriptionService: остання підписка студента на курс

        alt нової підписки немає і від end_date минуло > 7 діб
            SubscriptionService->>+Subscription: updateStatus(subscriptionId, "closed")
            Subscription-->>-SubscriptionService: статус оновлено
            Note right of Subscription: Homework_submission при цьому не змінюється
        else інакше
            Note right of SubscriptionService: статус не змінюється
        end
    end
    SubscriptionService-->>-Scheduler: checkCompleted()

    Note over Student,Homework_submission: Фаза 2: Олена відкриває курс і свої результати після закриття доступу

    Student->>+SubscriptionService: openCourse(courseId)
    SubscriptionService->>+Subscription: findLatestSubscription(studentId, courseId)
    Subscription-->>-SubscriptionService: підписка зі статусом "closed"
    SubscriptionService-->>-Student: заняття і ДЗ, створені до end_date

    Student->>+SubscriptionService: getSubmissionResults(courseId)
    SubscriptionService->>+Subscription: findLatestSubscription(studentId, courseId)
    Subscription-->>-SubscriptionService: підписка зі статусом "closed" (end_date = 2026-10-01)

    opt статус підписки "closed" (доступ закрито, історія лишається доступною)
        SubscriptionService->>+Homework_submission: findGradedSubmissions(studentId, courseId, endDate)
        Homework_submission-->>-SubscriptionService: здачі зі статусом "graded": grade = 85, feedback = "Гарна робота"
    end

    SubscriptionService-->>-Student: оцінка 85 і фідбек "Гарна робота"

Фаза 1 (щоденна перевірка) така сама, як у REQ-32-closed: loop по прострочених підписках і alt для статусу closed. Бар SubscriptionService відкривається до loop і закривається після нього, тож не обривається в гілці. Нотатка «Homework_submission не змінюється» стоїть у гілці alt.
Фаза 2 має дві взаємодії: openCourse і getSubmissionResults. У другій є opt з викликом findGradedSubmissions до Homework_submission.
Усі activation bars парні (+ і -).
Де діаграма відходить від таблиці в spec.md

### Коментар: