## Побудова sequence-діаграми на REQ-32

### Промпт 1

Побудуй Mermaid sequenceDiagram на сценарій REQ-31. Постав уточнюючі питання якщо вони зʼявляться під час створення діаграми за [spec](spec.md)

### Вихід 1

Ось діаграма @REQ-32 за таблицею зі spec_1.md. Там, де spec неоднозначний, я вибрала варіант, найближчий до діаграми @REQ-31, і виніс питання нижче.

sequenceDiagram
    autonumber
    actor Student
    participant Scheduler
    participant SubscriptionService
    participant Subscription

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
        else інакше
            Note right of SubscriptionService: статус не змінюється
        end
    end
    SubscriptionService-->>-Scheduler: checkCompleted()

    Note over Student,Subscription: Фаза 2: Олена відкриває курс і здає ДЗ після закриття доступу

    Student->>+SubscriptionService: openCourse(courseId)
    SubscriptionService->>+Subscription: findLatestSubscription(studentId, courseId)
    Subscription-->>-SubscriptionService: підписка зі статусом "closed"
    opt статус підписки "closed"
        SubscriptionService->>SubscriptionService: buildNewSubscriptionOffer()
    end
    SubscriptionService-->>-Student: заняття і ДЗ, створені до end_date (+ пропозиція оформити нову підписку, якщо closed)

    Student->>+SubscriptionService: submitHomework(homeworkId, file)
    SubscriptionService->>+Subscription: findLatestSubscription(studentId, courseId)
    Subscription-->>-SubscriptionService: підписка зі статусом "closed"
    alt статус "closed"
        SubscriptionService-->>Student: помилка "доступ закрито"
    else статус "active" або "grace"
        SubscriptionService-->>Student: здачу ДЗ прийнято
    end
    deactivate SubscriptionService

Що я вирішила сама
Порядок у фазі 1. У spec рядок updateStatus стоїть перед findLatestSubscription, але логічно спершу перевіряється, чи є нова підписка. У діаграмі findLatestSubscription іде першим, усередині loop.
Фільтр у findExpired. Тут стоїть «статус active або grace», як у @REQ-31, @REQ-33 і @REQ-34, а не «не closed», як у рядку @REQ-32.
Activation bars. Смуга SubscriptionService на submitHomework закривається одним deactivate після end, тому не обривається в гілці.

Питання
1. Рядок findLatestSubscription у spec. Там написано, що сервіс «отримує closed». Але на момент цього виклику статус ще active або grace, а closed виставляється пізніше в alt. Чи правильно, що цей виклик повертає просто «останню підписку», як у діаграмі, а closed з'являється лише в openCourse? Якщо так, рядок у spec варто виправити.
2. Звідки заняття й ДЗ. У openCourse повертаються «заняття та ДЗ, створені до end_date», але в таблиці учасників немає сутності, яка їх віддає. Чи додаємо Lesson із викликом на кшталт getLessonsBefore(courseId, endDate)? Це також закрило б прогалину з @REQ-31.
3. alt у openCourse. Рядок у spec позначено як alt, але я зробила один opt для пропозиції й один return. Чи потрібна повноцінна гілка else (статус active або grace → усі заняття й ДЗ)? У @REQ-32 вона недосяжна, але вимога щодо фрагментів і так виконана через loop, alt і opt у фазі 1.
4. Гілка else у submitHomework. У spec описано лише відхилення для closed. Я додала гілку «прийнято» для узгодженості з @REQ-31. Лишаємо її чи залишаємо тільки помилку?
Виклик findLatestSubscription у submitHomework. У рядку @REQ-32 немає окремого кроку для читання підписки, хоча в @REQ-31 він є. Додаємо відповідний рядок у spec?

Дайте відповіді (навіть коротко, «1 так, 2 так» тощо), і я оновлю діаграму та підготую таблицю @REQ-32 з тими самими змінами.  

### Промпт 2

1. Виклик повертає останню підписку на курс, щоб перевірити, чи немає новішої.
2. Це поза межами сценарію, бо сценарій перевіряє, чи дозволено здати дз, а не як саме зберігається здача.
3. Роби opt. Гілка else недосяжна, бо сценарій описує лише closed.
Нехай в opt будуть обидва внутрішні кроки:

text
    opt статус підписки "closed"
        SubscriptionService->>SubscriptionService: filterContentUntil(endDate)
        SubscriptionService->>SubscriptionService: buildNewSubscriptionOffer()
    end
4. Лиши тільки помилку
5. Так, додай

### Вихід 2

Оновила діаграму за вашими відповідями: opt у openCourse має обидва внутрішні кроки, у submitHomework лишилася тільки помилка, Lesson і збереження здачі не додавала.

sequenceDiagram
    autonumber
    actor Student
    participant Scheduler
    participant SubscriptionService
    participant Subscription

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
        else інакше
            Note right of SubscriptionService: статус не змінюється
        end
    end
    SubscriptionService-->>-Scheduler: checkCompleted()

    Note over Student,Subscription: Фаза 2: Олена відкриває курс і здає ДЗ після закриття доступу

    Student->>+SubscriptionService: openCourse(courseId)
    SubscriptionService->>+Subscription: findLatestSubscription(studentId, courseId)
    Subscription-->>-SubscriptionService: підписка зі статусом "closed"
    opt статус підписки "closed"
        SubscriptionService->>SubscriptionService: filterContentUntil(endDate)
        SubscriptionService->>SubscriptionService: buildNewSubscriptionOffer()
    end
    SubscriptionService-->>-Student: заняття і ДЗ, створені до end_date, і пропозиція оформити нову підписку

    Student->>+SubscriptionService: submitHomework(homeworkId, file)
    SubscriptionService->>+Subscription: findLatestSubscription(studentId, courseId)
    Subscription-->>-SubscriptionService: підписка зі статусом "closed"
    opt статус підписки "closed"
        SubscriptionService-->>Student: помилка "доступ закрито"
    end
    deactivate SubscriptionService

### Коментар
