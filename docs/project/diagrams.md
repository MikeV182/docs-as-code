# Диаграммы

## Диаграмма последовательности

Диаграмма показывает выполнение сценария
[UC-01. Классификация и направление обращения](requirements.md#uc-01).

```mermaid
sequenceDiagram
    actor User as Пользователь
    participant System as Система
    participant Model as ML-модель
    participant Queue as Очереди обращений
    actor Operator as Оператор поддержки

    User->>System: Отправить текст обращения
    System->>System: Сохранить обращение, присвоить номер
    System->>Model: Передать текст обращения
    Model-->>System: Вернуть категорию и уверенность

    alt Уверенность выше порога
        System->>Queue: Поместить в очередь группы по категории
        System-->>User: Показать номер и категорию
        Queue-->>Operator: Обращение в очереди группы
    else Модель не справилась или уверенность ниже порога
        System->>Queue: Поместить в общую очередь
        System-->>User: Показать номер и статус «Требует ручной классификации»
        Queue-->>Operator: Обращение в общей очереди
        Operator->>System: Назначить категорию
        System->>Queue: Переместить в очередь группы по категории
    end
```

## Диаграмма классов

Диаграмма показывает основные сущности сервиса и связи между ними.

```mermaid
classDiagram
    class User {
        +int id
        +string name
        +string email
    }

    class Operator {
        +int id
        +string name
    }

    class SupportGroup {
        +int id
        +string name
        +string category
    }

    class SupportRequest {
        +int id
        +string text
        +string status
        +string category
        +bool manuallyAssigned
    }

    class ClassificationResult {
        +string category
        +float confidence
    }

    User "1" --> "*" SupportRequest : отправляет
    SupportRequest "1" --> "0..1" ClassificationResult : результат модели
    SupportRequest "*" --> "0..1" SupportGroup : находится в очереди
    Operator "*" --> "1" SupportGroup : состоит в
```

Пояснения к диаграмме:

- **SupportGroup** — группа поддержки с собственной очередью. Каждая группа
  отвечает за одну категорию: «Оплата», «Доступ», «Ошибка» или «Вопрос».
- **SupportRequest** — обращение. Если обращение не привязано к группе,
  оно находится в общей очереди и имеет статус «Требует ручной классификации».
  Флаг `manuallyAssigned` показывает, что категорию назначил оператор (US-03).
- **ClassificationResult** — ответ ML-модели. Его может не быть, если модель
  была недоступна (сценарий 3a в UC-01).
