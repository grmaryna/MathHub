# Завдання 3 — Взаємодія (Sequence Diagram)

## UC-29: Закриття доступу до курсу після завершення підписки
Сценарії, описані у завданні 2 лежать у файлі [UC-29](UC-29.feature)

@UC29
**Функціонал: Закриття доступу до курсу після завершення підписки**

```feature
  Передумова:
    Припустимо існує курс "Підготовка до НМТ"
    І у курсі є заняття з посиланням і ДЗ з терміном здачі
    І учень Олена мала підписку на цей курс з end_date "2026-10-01"
    І Олена здала ДЗ, і її здача має статус "graded", оцінку 85 і фідбек "Гарна робота"
```

### Учасники сценаріїв
| Учасник | Тип | Роль |
|--|--|--|
| `Student` | актор | відкриває курс, здає дз |
| `Scheduler` | внутрішня система | щодня запускає перевірку підписок |
| `Subscription` | сутність | зберігає `end_date` і `status` |
| `SubscriptionService` | внутрішня сиситема | застосовує правила `grace`- періоду, змінює статус підписки, відповідає на запити `Student` |
| `Homework_submission` | сутність | зберігає здачу, `grade`, `feedback`, `graded_at`; закриття доступу їх не змінює |
| `Course` | сутність | надає `duration` і `price` для розрахунку нової підписки |
| `Payment Gateway` | зовнішня система | підтверджує оплату продовження |

## @REQ-31

```feature
  Структура сценарію: Доступ зберігається в межах grace-періоду
    Припустимо нової підписки на цей курс у Олени немає
    І від end_date минуло <днів> діб
    Коли Scheduler виконує щоденну перевірку підписок
    І Олена відкриває курс
    І Олена намагається здати ДЗ
    Тоді Олена має доступ до занять курсу
    І Олена бачить пропозицію продовжити підписку
    І Олена може здати ДЗ
```

### Кроки сценарію

| Взаємодія | Виклик | Що відбувається | Крок Gherkin |
|--|--|--|--|
| `Scheduler` - `SubscriptionService` | `runDailyCheck()` | `Scheduler` запускає щоденну перевірку підписок; після обробки всіх підписок `SubscriptionService` повертає `checkCompleted()` | Коли `Scheduler` виконує щоденну перевірку підписок |
| `SubscriptionService` - `Subscription` | `findExpired(today)` | `SubscriptionService` запитує підписки, у яких `end_date` уже минув і статус `active` або `grace`, і отримує їх список | І від `end_date` минуло <днів> діб |
| `SubscriptionService` - `Subscription` | `findLatestSubscription(studentId, courseId)` | loop: для кожної простроченої підписки `SubscriptionService` шукає останню підписку цього студента на цей курс, щоб перевірити, чи немає нової | Припустимо нової підписки на цей курс у Олени немає |
| `SubscriptionService` - `Subscription` | `updateStatus(subscriptionId, "grace")` | alt: якщо нової підписки немає і від `end_date` минуло не більше 7 діб, `SubscriptionService` виставляє статус `grace` (для підписки, що вже в `grace`, статус не змінюється); інакше статус не змінюється | — (внутрішній крок, забезпечує результат «Олена має доступ до занять курсу») |
| `Student` - `SubscriptionService` | `openCourse(courseId)` | `Student` відкриває курс; відповідь містить посилання на заняття і (якщо статус `grace`) пропозицію продовжити підписку | Тоді Олена має доступ до занять курсу |
| `SubscriptionService` - `Subscription` | `findLatestSubscription(studentId, courseId)` | `SubscriptionService` читає останню підписку студента на цей курс і отримує статус `grace` | — (внутрішній крок: джерело статусу для доступу до курсу) |
| `SubscriptionService` - `SubscriptionService` | `buildRenewalOffer()` | opt: якщо статус `grace`, `SubscriptionService` формує пропозицію продовжити підписку для відповіді | І Олена бачить пропозицію продовжити підписку |
| `Student` - `SubscriptionService` | `submitHomework(homeworkId, file)` | `Student` здає ДЗ; `SubscriptionService` повертає «здачу прийнято» після перевірки статусу підписки | І Олена може здати ДЗ |
| `SubscriptionService` - `Subscription` | `findLatestSubscription(studentId, courseId)` | `SubscriptionService` читає останню підписку студента на цей курс і отримує статус `grace`, тому здачу приймають | — (внутрішній крок: джерело статусу для прийняття здачі) |

## @REQ-32-closed

```feature
Сценарій: Доступ закривається після 7 діб без продовження
  Припустимо нової підписки на цей курс у Олени немає
  І від end_date минуло 8 діб
  І після end_date у курсі опубліковано нове заняття і нове ДЗ
  Коли Scheduler виконує щоденну перевірку підписок
  Тоді Олена не бачить нове заняття і нове ДЗ, опубліковані після end_date
  І Олена бачить заняття і ДЗ, опубліковані до end_date
  І Олена бачить пропозицію оформити нову підписку
  І Олена не може здати ДЗ попередніх занять
```

### Кроки сценарію

| Взаємодія | Виклик | Що відбувається | Крок Gherkin |
|--|--|--|--|
| `Scheduler` - `SubscriptionService` | `runDailyCheck()` | `Scheduler` запускає щоденну перевірку підписок; після обробки всіх підписок `SubscriptionService` повертає `checkCompleted()` | Коли `Scheduler` виконує щоденну перевірку підписок |
| `SubscriptionService` - `Subscription` | `findExpired(today)` | `SubscriptionService` запитує підписки, у яких `end_date` уже минув і статус `active` або `grace`, і отримує їх список | І від `end_date` минуло 8 діб |
| `SubscriptionService` - `Subscription` | `findLatestSubscription(studentId, courseId)` | loop: для кожної простроченої підписки `SubscriptionService` шукає останню підписку цього студента на цей курс, щоб перевірити, чи немає нової | Припустимо нової підписки на цей курс у Олени немає |
| `SubscriptionService` - `Subscription` | `updateStatus(subscriptionId, "closed")` | alt: якщо нової підписки немає і від `end_date` минуло більше 7 діб, `SubscriptionService` виставляє статус `closed`; інакше статус не змінюється | внутрішній крок |
| `Student` - `SubscriptionService` | `openCourse(courseId)` | `Student` відкриває курс; відповідь містить заняття і ДЗ, створені до `end_date`, та пропозицію оформити нову підписку | Тоді Олена не бачить нове заняття і нове ДЗ, опубліковані після `end_date` |
| `SubscriptionService` - `Subscription` | `findLatestSubscription(studentId, courseId)` | `SubscriptionService` читає останню підписку студента на цей курс і отримує статус `closed` | внутрішній крок |
| `SubscriptionService` - `SubscriptionService` | `filterContentUntil(endDate)` | opt (один фрагмент із наступним рядком): якщо статус `closed`, залишає лише заняття та ДЗ, створені до `end_date` | І Олена бачить заняття і ДЗ, опубліковані до `end_date` |
| `SubscriptionService` - `SubscriptionService` | `buildNewSubscriptionOffer()` | opt: якщо статус `closed`, формує пропозицію оформити нову підписку для відповіді | І Олена бачить пропозицію оформити нову підписку |
| `Student` - `SubscriptionService` | `submitHomework(homeworkId, file)` | `Student` здає ДЗ; `SubscriptionService` повертає помилку "доступ закрито" після перевірки статусу підписки | І Олена не може здати ДЗ попередніх занять |
| `SubscriptionService` - `Subscription` | `findLatestSubscription(studentId, courseId)` | `SubscriptionService` читає останню підписку студента на цей курс і отримує статус `closed`, тому відхиляє здачу | внутрішній крок |

## @REQ-32-history

```feature
  Сценарій: Історія здач і оцінок зберігається після закриття доступу
    Припустимо нової підписки на цей курс у Олени немає
    І від end_date минуло 8 діб +
    Коли Scheduler виконує щоденну перевірку підписок
    І Олена відкриває курс
    І Олена відкриває свої результати
    Тоді Олена бачить заняття і ДЗ, опубліковані до end_date
    І Олена бачить оцінку 85 і фідбек "Гарна робота" до ДЗ
```

### Кроки сценарію

| Взаємодія | Виклик | Що відбувається | Крок Gherkin |
|--|--|--|--|
| `Scheduler` - `SubscriptionService` | `runDailyCheck()` | `Scheduler` запускає щоденну перевірку підписок | Коли `Scheduler` виконує щоденну перевірку підписок |
| `SubscriptionService` - `Subscription` | `findExpired(today)` | `SubscriptionService` запитує підписки, у яких `end_date` уже минув і статус `active` або `grace`, і отримує їх список | І від `end_date` минуло 8 діб |
| `SubscriptionService` - `Subscription` | `findLatestSubscription(studentId, courseId)` | loop: для кожної простроченої підписки `SubscriptionService` шукає останню підписку цього студента на цей курс, щоб перевірити, чи немає нової | І нової підписки на цей курс у Олени немає |
| `SubscriptionService` - `Subscription` | `updateStatus(subscriptionId, "closed")` | alt: якщо нової підписки немає і від `end_date` минуло більше 7 діб, `SubscriptionService` виставляє статус `closed`. `Homework_submission` при цьому не змінюється | внутрішня робота |
| `Student` - `SubscriptionService` | `openCourse(courseId)` | `Student` відкриває курс, `SubscriptionService` читає останню підписку, бачить статус `closed` і повертає лише заняття та ДЗ, створені до `end_date` | Тоді Олена бачить заняття і ДЗ, опубліковані до `end_date` |
| `Student` - `SubscriptionService` | `getSubmissionResults(courseId)` | `Student` просить свої результати за курсом; статус підписки не перевіряється, історія доступна за будь-якого статусу | І Олена бачить оцінку 85 і фідбек "Гарна робота" до ДЗ |
| `SubscriptionService` - `Homework_submission` | `findGradedSubmissions(studentId, courseId)` | `SubscriptionService` читає здачі студента за курсом зі статусом `graded` і повертає `grade` і `feedback`; фільтра за датою немає | І Олена бачить оцінку 85 і фідбек "Гарна робота" до ДЗ |

## @REQ-34

```feature
  Сценарій: Продовження в grace-період зберігає доступ
    Припустимо від end_date минуло 3 доби
    Коли платіжний шлюз підтверджує оплату продовження підписки
    І Scheduler виконує щоденну перевірку підписок
    І Олена відкриває курс
    Тоді Олена має доступ до занять курсу
    І створено нову підписку, що починається "2026-10-02"
    І доступ за попередньою підпискою не закрито
```

### Кроки сценарію
| Взаємодія | Виклик | Що відбувається | Крок Gherkin |
|--|--|--|--|
| `Payment Gateway` - `SubscriptionService` | `confirmPayment(studentId, courseId, paymentMethod)` | `Payment Gateway` повідомляє про успішну оплату продовження | Коли платіжний шлюз підтверджує оплату продовження підписки |
| `SubscriptionService` - `Subscription` | `findLatestSubscription(studentId, courseId)` | `SubscriptionService` знаходить попередню підписку Олени (`end_date` = 2026-10-01, статус `grace`) | І учень Олена мала підписку на цей курс з end_date "2026-10-01" (передумова) |
| `SubscriptionService` - `Course` | `getCourseTerms(courseId)` | `SubscriptionService` отримує `duration` і поточну `price` курсу | внутрішній крок |
| `SubscriptionService` - `Subscription` | `createSubscription(studentId, courseId, startDate, endDate, paymentMethod, price)` | `SubscriptionService` створює нову підписку: `startDate` = попередній `end_date` + 1 доба, статус `active` | І створено нову підписку, що починається "2026-10-02" |
| `Scheduler` - `SubscriptionService` | `runDailyCheck()` | `Scheduler` запускає щоденну перевірку підписок | І `Scheduler` виконує щоденну перевірку підписок |
| `SubscriptionService` - `Subscription` | `findExpired(today)` | `SubscriptionService` запитує підписки, у яких `end_date` уже минув і статус `active` або `grace`; у списку є попередня підписка Олени (статус `grace`) | Припустимо від end_date минуло 3 доби |
| `SubscriptionService` - `Subscription` | `findLatestSubscription(studentId, courseId)` | loop: для кожної простроченої підписки `SubscriptionService` перевіряє, чи є новіша підписка | внутрішній крок |
| `SubscriptionService` - `Subscription` | `updateStatus(subscriptionId, "expired")` | alt: якщо є новіша підписка, `SubscriptionService` позначає стару як `expired` і не закриває доступ; інакше (нової немає) діють сценарії 1 і 2 | Тоді Олена має доступ до занять курсу |
| `Student` - `SubscriptionService` | `openCourse(courseId)` | `Student` відкриває курс, `SubscriptionService` читає останню підписку (нову, `active`) і повертає посилання на заняття | Тоді Олена має доступ до занять курсу |
| `SubscriptionService` - `Student` | `returnCourseAccess(lessonLinks)` | opt: пропозиція продовжити додається лише для `grace`/`closed`; для `active` її немає | І Олена не бачить пропозиції продовжити підписку |