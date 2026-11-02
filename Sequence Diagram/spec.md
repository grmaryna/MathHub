# Завдання 3 — Взаємодія (Sequence Diagram)

## UC-29: Закриття доступу до курсу після завершення підписки
Сценарії, описані у завданні 2 лежать у файлі [UC-29](UC-29.feature)

## @REQ-31

  Структура сценарію: Доступ зберігається в межах grace-періоду
    Припустимо нової підписки на цей курс у Олени немає
    І від end_date минуло <днів> діб
    Коли Scheduler виконує щоденну перевірку підписок
    Тоді Олена має доступ до занять курсу
    І Олена може здати ДЗ
    І Олена бачить пропозицію продовжити підписку

### Учасники
| Учасник | Тип | Роль |
|--|--|--|
| `Student` | актор | відкриває курс, здає дз |
| `Scheduler` | внутрішня система | щодня запускає перевірку підписок |
| `Subscription` | сутність | зберігає `end_date` і `status` |
| `SubscriptionService` | внутрішня сиситема | застосовує правила grace-періоду, змінює статус підписки, відповідає на запити Student |


### Кроки сценарію
| Взаємодія | Виклик | Що відбувається | Крок Gherkin |
|--|--|--|--|
| `Scheduler` - `SubscriptionService` | `runDailyCheck()` | `Scheduler` запускає щоденну перевірку підписок | Коли `Scheduler` виконує щоденну перевірку підписок |
| `SubscriptionService` - `Subscription` | `findExpired(today)` | `SubscriptionService` запитує всі підписки, у яких `end_date` уже минув, і отримує їх список | І від `end_date` минуло <днів> діб |
| `SubscriptionService` - `Subscription` | `findLatestSubscription(studentId, courseId)` | loop: для кожної прострочена підписки `SubscriptionService` шукає останню підписку цього студента на цей курс, щоб перевірити, чи немає нової | Припустимо нової підписки на цей курс у Олени немає |
| `SubscriptionService` - `Subscription` | `updateStatus(subscriptionId, “grace”)` | alt: якщо нової підписки немає і від `end_date` минуло не більше 7 діб, `SubscriptionService` виставляє статус `grace` | внутрішній крок |
| `Student` - `SubscriptionService` | `openCourse(courseId)` | Коли `Student` відкриває курс, то `SubscriptionService` читає останню підписку, бачить статус `grace` і повертає посилання на заняття | Тоді Олена має доступ до занять курсу |
| `Student` - `SubscriptionService` | `submitHomework(homeworkId, file)` | Student здає ДЗ, `SubscriptionService` перевіряє, що підписка має статус `active` або `grace`, і приймає здачу | І Олена може здати ДЗ |
| `SubscriptionService` - `Student` | `returnRenewalOffer()` | Якщо статус підписки `grace`, `SubscriptionService` додає до відповіді пропозицію продовжити підписку | І Олена бачить пропозицію продовжити підписку |

## @REQ-32

Сценарій: Доступ закривається після 7 діб без продовження
  Припустимо нової підписки на цей курс у Олени немає
  І від end_date минуло 8 діб
  І після end_date у курсі опубліковано нове заняття і нове ДЗ
  Коли Scheduler виконує щоденну перевірку підписок
  Тоді Олена не бачить нове заняття і нове ДЗ, опубліковані після end_date
  І Олена бачить заняття і ДЗ, опубліковані до end_date
  І Олена не може здати ДЗ попередніх занять
  І Олена бачить пропозицію оформити нову підписку

### Учасники
| Учасник | Тип | Роль |
|--|--|--|
| `Student` | актор | відкриває курс, здає дз |
| `Scheduler` | внутрішня система | щодня запускає перевірку підписок |
| `Subscription` | сутність | зберігає `end_date` і `status` |
| `SubscriptionService` | внутрішня сиситема | застосовує правила `grace`-періоду, змінює статус підписки, відповідає на запити `Student` |

### Кроки сценарію
| Взаємодія | Виклик | Що відбувається | Крок Gherkin |
|--|--|--|--|
| `Scheduler` - `SubscriptionService` | `runDailyCheck()` | `Scheduler` запускає щоденну перевірку підписок | Коли `Scheduler` виконує щоденну перевірку підписок |
| `SubscriptionService` - `Subscription` | `findExpired(today)` | `SubscriptionService` запитує підписки, у яких `end_date` уже минув і статус не `closed`, і отримує їх список | І від `end_date` минуло 8 діб |
| `SubscriptionService` - `Subscription` | `findLatestSubscription(studentId, courseId)` | loop: для кожної простроченої підписки `SubscriptionService` шукає останню підписку цього студента на цей курс, щоб перевірити, чи немає нової | Припустимо нової підписки на цей курс у Олени немає |
| `SubscriptionService` - `Subscription` | `updateStatus(subscriptionId, "closed")` | alt: якщо нової підписки немає і від `end_date` минуло більше 7 діб, `SubscriptionService` виставляє статус `closed` | внутрішній крок |
| `Student` - `SubscriptionService` | `openCourse(courseId)` | alt: `Student` відкриває курс, `SubscriptionService` читає останню підписку, бачить статус `closed` і повертає лише заняття та ДЗ, створені до `end_date` | Тоді Олена не бачить нове заняття і нове ДЗ, опубліковані після `end_date` / І Олена бачить заняття і ДЗ, опубліковані до `end_date` |
| `Student` - `SubscriptionService` | `submitHomework(homeworkId, file)` | `Student` здає ДЗ, `SubscriptionService` перевіряє статус підписки; якщо `closed`, відхиляє здачу з помилкою | І Олена не може здати ДЗ попередніх занять |
| `SubscriptionService` - `Student` | `returnRenewalOffer()` | у відповіді на `openCourse` `SubscriptionService` додає пропозицію оформити нову підписку | І Олена бачить пропозицію оформити нову підписку |

