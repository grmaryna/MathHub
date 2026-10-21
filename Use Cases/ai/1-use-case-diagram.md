## Побудова Use Case діаграми

### Промпт

Побудуй Use Case діаграму (Mermaid) на основі даного [spec.md](spec.md)

### Вихід

flowchart LR
  U["User"]:::human
  G["Guest"]:::human
  S["Student"]:::human
  P["Parent"]:::human
  T["Teacher"]:::human
  A["Admin"]:::human
  PG{{"Payment Gateway"}}:::system
  ES{{"Email Service"}}:::system
  SCH{{"Scheduler"}}:::system

  S -->|"is-a"| U
  P -->|"is-a"| U
  T -->|"is-a"| U
  A -->|"is-a"| U

  subgraph SYS["Платформа онлайн-навчання"]
    UC1(["Зареєструватися"]):::uc
    UC2(["Підтвердити email"]):::uc
    UC3(["Автентифікуватися"]):::uc
    UC4(["Змінити пароль"]):::uc
    UC5(["Створити акаунт учня"]):::uc
    UC6(["Переглядати прогрес дітей"]):::uc
    UC7(["Оформити підписку"]):::uc
    UC8(["Оплатити підписку"]):::uc
    UC9(["Застосувати промокод"]):::uc
    UC10(["Продовжити підписку"]):::uc
    UC11(["Відвідувати заняття"]):::uc
    UC12(["Здати ДЗ"]):::uc
    UC13(["Повторно здати ДЗ"]):::uc
    UC14(["Переглянути оцінку та фідбек"]):::uc
    UC15(["Редагувати власний курс"]):::uc
    UC16(["Додати заняття"]):::uc
    UC17(["Додати запис заняття"]):::uc
    UC18(["Призначити ДЗ"]):::uc
    UC19(["Переглянути неоцінені ДЗ"]):::uc
    UC20(["Оцінити роботу та дати фідбек"]):::uc
    UC21(["Керувати курсами"]):::uc
    UC22(["Зареєструвати викладача"]):::uc
    UC23(["Деактивувати викладача"]):::uc
    UC24(["Керувати промокодами"]):::uc
    UC25(["Надіслати нагадування про закінчення підписки"]):::uc
    UC26(["Закрити доступ після grace-періоду"]):::uc
    UC27(["Надіслати email-сповіщення"]):::uc
  end

  %% Асоціації
  G --- UC1
  G --- UC2
  G --- UC3
  U --- UC3
  U --- UC4
  P --- UC5
  P --- UC6
  P --- UC7
  S --- UC7
  S --- UC11
  S --- UC12
  S --- UC14
  T --- UC15
  T --- UC16
  T --- UC17
  T --- UC18
  T --- UC19
  T --- UC20
  A --- UC21
  A --- UC22
  A --- UC23
  A --- UC24
  PG --- UC8
  ES --- UC27
  SCH --- UC25
  SCH --- UC26

  %% include: базовий UC -> включений
  UC1 -.->|"«include»"| UC2
  UC1 -.->|"«include»"| UC27
  UC7 -.->|"«include»"| UC8
  UC18 -.->|"«include»"| UC27
  UC20 -.->|"«include»"| UC27
  UC22 -.->|"«include»"| UC27
  UC25 -.->|"«include»"| UC27

  %% extend: розширення -> базовий UC
  UC9 -.->|"«extend»"| UC7
  UC10 -.->|"«extend»"| UC7
  UC13 -.->|"«extend»"| UC12

  linkStyle 0,1,2,3 stroke-width:3px
  classDef human fill:#EEEDFE,stroke:#534AB7,color:#3C3489
  classDef system fill:#F1EFE8,stroke:#5F5E5A,color:#444441
  classDef uc fill:#E1F5EE,stroke:#0F6E56,color:#085041

  