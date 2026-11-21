## Побудова sequence-діаграми на REQ-32-history

### Промпт

Побудуй Mermaid sequenceDiagram на сценарій REQ-46 за [spec](spec.md)

### Вихід

Діаграму для REQ-46 я створила: diagram-req-46.mmd. Побудувала її за таблицею кроків колишнього REQ-34 зі spec.md. Рендер не перевіряла, тож встав код у Mermaid Live Editor.

sequenceDiagram
    autonumber
    actor Student
    participant PaymentGateway as Payment Gateway
    participant Scheduler
    participant SubscriptionService
    participant Subscription
    participant Course

    Note over PaymentGateway,Course: Фаза 1: оплата продовження в grace-період (від end_date минуло 3 доби)

    PaymentGateway->>+SubscriptionService: confirmPayment(studentId, courseId, paymentMethod)
    SubscriptionService->>+Subscription: findLatestSubscription(studentId, courseId)
    Subscription-->>-SubscriptionService: попередня підписка (end_date = 2026-10-01, статус "grace")
    SubscriptionService->>+Course: getCourseTerms(courseId)
    Course-->>-SubscriptionService: duration і поточна price курсу

    opt є попередня підписка в статусі "grace"
        Note right of SubscriptionService: startDate = попередній end_date + 1 доба = 2026-10-02
    end

    SubscriptionService->>+Subscription: createSubscription(studentId, courseId, startDate, endDate, paymentMethod, price)
    Subscription-->>-SubscriptionService: нова підписка (start_date = 2026-10-02, статус "active")
    SubscriptionService-->>-PaymentGateway: оплату підтверджено

    Note over Scheduler,Subscription: Фаза 2: щоденна перевірка підписок

    Scheduler->>+SubscriptionService: runDailyCheck()
    SubscriptionService->>+Subscription: findExpired(today)
    Subscription-->>-SubscriptionService: список прострочених підписок (серед них попередня підписка Олени, статус "grace")

    loop для кожної простроченої підписки
        SubscriptionService->>+Subscription: findLatestSubscription(studentId, courseId)
        Subscription-->>-SubscriptionService: остання підписка студента на курс

        alt є новіша підписка
            SubscriptionService->>+Subscription: updateStatus(subscriptionId, "expired")
            Subscription-->>-SubscriptionService: статус оновлено
            Note right of Subscription: стара підписка позначена expired, доступ не закрито
        else нової підписки немає
            Note right of SubscriptionService: діють сценарії REQ-31 і REQ-32
        end
    end
    SubscriptionService-->>-Scheduler: checkCompleted()

    Note over Student,Subscription: Фаза 3: Олена відкриває курс після продовження

    Student->>+SubscriptionService: openCourse(courseId)
    SubscriptionService->>+Subscription: findLatestSubscription(studentId, courseId)
    Subscription-->>-SubscriptionService: нова підписка зі статусом "active"

    opt статус підписки "grace" або "closed"
        SubscriptionService->>SubscriptionService: buildRenewalOffer()
    end

    SubscriptionService-->>-Student: посилання на заняття курсу (без пропозиції продовжити, бо статус "active")

Структура
Фаза 1: оплата. Payment Gateway викликає confirmPayment, SubscriptionService читає попередню підписку (статус grace), бере умови курсу (getCourseTerms) і створює нову підписку зі start_date 2026-10-02 (createSubscription).
Фаза 2: щоденна перевірка. loop по прострочених підписках і alt: якщо є новіша підписка, стара отримує статус expired, доступ не закривається; інакше діють REQ-31 і REQ-32.
Фаза 3: відкриття курсу. openCourse читає нову підписку (active) і повертає посилання на заняття; opt з buildRenewalOffer() для grace або closed у цьому сценарії не спрацьовує.

Activation bars парні, а бар SubscriptionService не обривається в гілках alt і opt. Додані фрагменти: opt, loop, alt.

### Коментар:
