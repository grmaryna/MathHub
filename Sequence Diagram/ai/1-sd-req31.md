## Побудова sequence-діаграми на REQ-31 

### Промпт 1

Побудуй Mermaid sequenceDiagram на сценарій REQ-31. Постав уточнюючі питання якщо вони зʼявляться під час створення діаграми за [spec](spec.md)

### Вихід 1

Ось діаграма для REQ-31. Дії Student відбуваються після щоденної перевірки, тому я розділив їх нотатками на дві фази.

sequenceDiagram
    autonumber
    actor Student
    participant Scheduler
    participant SubscriptionService
    participant Subscription

    Note over Scheduler,Subscription: Фаза 1: щоденна перевірка підписок

    Scheduler->>SubscriptionService: runDailyCheck()
    SubscriptionService->>Subscription: findExpired(today)
    Subscription-->>SubscriptionService: список прострочених підписок (status ≠ closed)

    loop для кожної простроченої підписки
        SubscriptionService->>Subscription: findLatestSubscription(studentId, courseId)
        Subscription-->>SubscriptionService: остання підписка студента на курс

        alt нової підписки немає і від end_date минуло ≤ 7 діб
            SubscriptionService->>Subscription: updateStatus(subscriptionId, "grace")
            Subscription-->>SubscriptionService: статус оновлено
        end
    end

    Note over Student,Subscription: Фаза 2: Олена користується курсом у grace-періоді

    Student->>SubscriptionService: openCourse(courseId)
    SubscriptionService->>Subscription: findLatestSubscription(studentId, courseId)
    Subscription-->>SubscriptionService: підписка зі статусом "grace"
    SubscriptionService-->>Student: посилання на заняття курсу

    Student->>SubscriptionService: submitHomework(homeworkId, file)
    Note right of SubscriptionService: перевіряє, що статус "active" або "grace"
    SubscriptionService-->>Student: здачу ДЗ прийнято

    opt статус підписки "grace"
        SubscriptionService-->>Student: returnRenewalOffer() — пропозиція продовжити підписку
    end

Припущення, які я зробив:

У кроці openCourse додав виклик findLatestSubscription до Subscription, бо в таблиці сказано, що сервіс «читає останню підписку».
returnRenewalOffer() показав як opt усередині відповіді сервісу, а не як окремий виклик.
Гілку else (більше 7 діб, статус closed) не додавав, бо вона не входить у REQ-31.

Уточнювальні питання:

1. Homework_submission. У таблиці submitHomework приймає лише SubscriptionService, але здача десь зберігається. Додати Homework_submission як учасника (наприклад, saveSubmission(...)), чи це поза межами цього сценарію?
2. Course. Звідки беруться посилання на заняття в openCourse: від SubscriptionService напряму чи через Course? Зараз Course не показано.
3. Межа 7 діб. Чи входить сьомий день у grace-період (≤ 7), чи ні (< 7)? Це важливо для прикладів у таблиці <днів>.
4. Повторні запуски. findExpired повертає всі підписки зі статусом не closed, тобто й ті, що вже в grace. Чи потрібно в діаграмі показувати, що updateStatus для них пропускається або є ідемпотентним?
Нова підписка. Що відбувається, якщо findLatestSubscription знаходить нову підписку (гілка, коли alt не виконується)? Чи треба показати цей випадок?

Скажіть, що з цього уточнити, і я оновлю діаграму.

### Промпт 2

Відповіді на питання:
1. Це поза межами сценарію, бо сценарій перевіряє, чи дозволено здати дз, а не як саме зберігається здача.
2. SubscriptionService бере посилання на заняття напряму з Lesson.link
3. 7 день входить у grace-період
4. Повторні запуски нічого не ламають: для підписок, що вже в grace, updateStatus виставляє той самий статус (операція ідемпотентна), closed і expired у вибірку findExpired не потрапляють
5. Якщо findLatestSubscription знаходить нову підписку, умова alt хибна, тому спрацьовує гілка else («статус не змінюється»)

### Відповідь 2

Дякую, оновив діаграму за вашими відповідями.

sequenceDiagram
    autonumber
    actor Student
    participant Scheduler
    participant SubscriptionService
    participant Subscription
    participant Lesson

    Note over Scheduler,Subscription: Фаза 1: щоденна перевірка підписок

    Scheduler->>SubscriptionService: runDailyCheck()
    SubscriptionService->>Subscription: findExpired(today)
    Subscription-->>SubscriptionService: список прострочених підписок (без closed і expired)

    loop для кожної простроченої підписки
        SubscriptionService->>Subscription: findLatestSubscription(studentId, courseId)
        Subscription-->>SubscriptionService: остання підписка студента на курс

        alt нової підписки немає і від end_date минуло ≤ 7 діб
            SubscriptionService->>Subscription: updateStatus(subscriptionId, "grace")
            Note right of Subscription: ідемпотентно: для підписки, що вже в grace, статус той самий
            Subscription-->>SubscriptionService: статус оновлено
        else знайдено нову підписку
            Note right of SubscriptionService: статус не змінюється
        end
    end

    Note over Student,Lesson: Фаза 2: Олена користується курсом у grace-періоді

    Student->>SubscriptionService: openCourse(courseId)
    SubscriptionService->>Subscription: findLatestSubscription(studentId, courseId)
    Subscription-->>SubscriptionService: підписка зі статусом "grace"
    SubscriptionService->>Lesson: getLink(courseId)
    Lesson-->>SubscriptionService: посилання на заняття
    SubscriptionService-->>Student: посилання на заняття курсу

    Student->>SubscriptionService: submitHomework(homeworkId, file)
    Note right of SubscriptionService: перевіряє, що статус "active" або "grace"
    SubscriptionService-->>Student: здачу ДЗ прийнято

    opt статус підписки "grace"
        SubscriptionService-->>Student: returnRenewalOffer() — пропозиція продовжити підписку
    end

Одне припущення: назву виклику getLink(courseId) я вигадав, бо в таблиці вона не вказана. Якщо в завданні 2 чи в моделі є інша назва, підставлю її.

### Коментар:
- у діаграмі немає `activation bars`, потрібно розділити стрілки на вииклики та відповіді;
- немає відповіді `Scheduler` на `runDailyCheck()`
- змінити підпис гілки else («знайдено нову підписку») треба змінити, бо він покриває і grace-період;
- пропозиція продовжити підписку має бути після відкривання курсу.
- прибрати `Lesson`, бо це напряму не стосується даного функціоналу 

### Виправлення:
