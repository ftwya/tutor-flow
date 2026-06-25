# TutorFlow — техническое задание

## 1. Назначение проекта

TutorFlow — веб-приложение для управления частной репетиторской практикой.

Приложение позволяет преподавателю:

* вести базу учеников;
* планировать занятия;
* учитывать проведённые и отменённые занятия;
* выдавать домашние задания;
* учитывать оплаты и задолженности;
* просматривать статистику доходов;
* находить информацию по ученикам;
* управлять данными через веб-интерфейс.

Проект разрабатывается как портфолио-проект PHP-разработчика и должен демонстрировать понимание backend-разработки, SQL, REST API, безопасности, клиент-серверного взаимодействия, Vue и работы с Git.

## 2. Технологический стек

### Backend

* PHP 8.4;
* PostgreSQL;
* PDO;
* Composer;
* PSR-4;
* PHPUnit;
* Nginx;
* Docker Compose.

### Frontend

* Vue 3;
* Vite;
* JavaScript;
* Fetch API;
* HTML;
* CSS.

### Инфраструктура

* Git;
* GitHub;
* GitHub Issues;
* GitHub Actions;
* Docker;
* Linux-сервер;
* HTTPS;
* переменные окружения.

## 3. Архитектура

Backend должен быть разделён на уровни:

```text
Router
↓
Middleware
↓
Controller
↓
Service
↓
Repository
↓
PostgreSQL
```

Ответственность компонентов:

### Router

* определяет маршрут;
* извлекает параметры;
* вызывает подходящий обработчик;
* не содержит бизнес-логику;
* не выполняет SQL.

### Middleware

* проверяет авторизацию;
* проверяет формат запроса;
* может обрабатывать CORS;
* не содержит бизнес-логику предметной области.

### Controller

* принимает HTTP-запрос;
* извлекает входные данные;
* вызывает Service;
* формирует HTTP-ответ;
* не выполняет SQL;
* не содержит сложную бизнес-логику.

### Service

* содержит бизнес-правила;
* проверяет допустимость операций;
* управляет несколькими Repository;
* выбрасывает предметные исключения.

### Repository

* работает с базой данных;
* использует подготовленные PDO-запросы;
* не формирует HTTP-ответы;
* не содержит HTML;
* не содержит бизнес-логику интерфейса.

## 4. Структура backend

Ориентировочная структура:

```text
backend/
├── public/
│   └── index.php
├── src/
│   ├── Controller/
│   ├── Service/
│   ├── Repository/
│   ├── Entity/
│   ├── DTO/
│   ├── Validation/
│   ├── Middleware/
│   ├── Exception/
│   └── Core/
├── config/
├── migrations/
├── tests/
├── bin/
├── composer.json
├── Dockerfile
└── .env.example
```

## 5. Структура frontend

```text
frontend/
├── src/
│   ├── api/
│   ├── components/
│   ├── views/
│   ├── router/
│   ├── stores/
│   ├── utils/
│   └── assets/
├── public/
├── package.json
├── vite.config.js
└── Dockerfile
```

На первом этапе Pinia допускается не использовать. Состояние можно хранить в компонентах и composables. Pinia добавляется только после появления реальной необходимости.

## 6. Пользователи и авторизация

В MVP приложение рассчитано на преподавателей.

Функции:

* регистрация;
* вход;
* выход;
* получение текущего пользователя;
* защита приватных маршрутов;
* хранение пароля через `password_hash`;
* проверка через `password_verify`;
* пользователь видит только принадлежащие ему данные.

Предпочтительный способ авторизации для MVP:

* PHP session;
* cookie с параметрами HttpOnly, Secure и SameSite;
* CSRF-защита для изменяющих запросов.

Не использовать самописное шифрование паролей.

## 7. Сущности

### User

```text
id
name
email
password_hash
created_at
updated_at
```

### Student

```text
id
user_id
full_name
grade
student_phone
parent_name
parent_phone
default_price
lesson_duration
format
subject
goal
notes
status
created_at
updated_at
```

Статусы:

```text
active
paused
archived
```

### Lesson

```text
id
user_id
student_id
scheduled_at
duration_minutes
price
status
topic
teacher_comment
created_at
updated_at
```

Статусы:

```text
planned
completed
cancelled_by_student
cancelled_by_teacher
missed
```

### Homework

```text
id
user_id
student_id
lesson_id
description
deadline
status
teacher_comment
created_at
updated_at
```

Статусы:

```text
assigned
completed
not_completed
checked
```

### Payment

```text
id
user_id
student_id
lesson_id
amount
status
payment_method
paid_at
comment
created_at
updated_at
```

Статусы:

```text
pending
paid
cancelled
```

## 8. Связи

```text
users 1:N students
users 1:N lessons
users 1:N payments
users 1:N homeworks

students 1:N lessons
students 1:N payments
students 1:N homeworks

lessons 1:N homeworks
lessons 1:0..1 payments
```

Все запросы к пользовательским данным должны учитывать `user_id`.

Пользователь не должен получить доступ к чужой записи, даже если вручную изменит ID в URL.

## 9. API

Базовый путь:

```text
/api/v1
```

### Auth

```text
POST   /api/v1/auth/register
POST   /api/v1/auth/login
POST   /api/v1/auth/logout
GET    /api/v1/auth/me
```

### Students

```text
GET    /api/v1/students
POST   /api/v1/students
GET    /api/v1/students/{id}
PUT    /api/v1/students/{id}
DELETE /api/v1/students/{id}
PATCH  /api/v1/students/{id}/status
```

Фильтры:

```text
search
grade
status
page
per_page
sort
direction
```

### Lessons

```text
GET    /api/v1/lessons
POST   /api/v1/lessons
GET    /api/v1/lessons/{id}
PUT    /api/v1/lessons/{id}
DELETE /api/v1/lessons/{id}
PATCH  /api/v1/lessons/{id}/status
```

Фильтры:

```text
student_id
status
date_from
date_to
page
per_page
```

### Homeworks

```text
GET    /api/v1/homeworks
POST   /api/v1/homeworks
GET    /api/v1/homeworks/{id}
PUT    /api/v1/homeworks/{id}
DELETE /api/v1/homeworks/{id}
PATCH  /api/v1/homeworks/{id}/status
```

### Payments

```text
GET    /api/v1/payments
POST   /api/v1/payments
GET    /api/v1/payments/{id}
PUT    /api/v1/payments/{id}
DELETE /api/v1/payments/{id}
PATCH  /api/v1/payments/{id}/status
```

### Dashboard

```text
GET /api/v1/dashboard/summary
GET /api/v1/dashboard/income
GET /api/v1/dashboard/debts
```

## 10. Формат ответа

Успешный ответ:

```json
{
  "data": {},
  "meta": {}
}
```

Ответ с ошибкой:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Переданы некорректные данные",
    "fields": {
      "full_name": [
        "Поле обязательно"
      ]
    }
  }
}
```

## 11. HTTP-статусы

Использовать корректные коды:

```text
200 OK
201 Created
204 No Content
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Unprocessable Entity
500 Internal Server Error
```

## 12. Валидация

Backend обязан проверять все входные данные.

Frontend-валидация используется только для удобства пользователя и не заменяет backend-валидацию.

Проверять:

* обязательные поля;
* длину строк;
* email;
* допустимые статусы;
* числовые диапазоны;
* даты;
* принадлежность связанных объектов пользователю;
* существование связанных записей.

## 13. Безопасность

Обязательные требования:

* подготовленные PDO-запросы;
* защита от SQL-инъекций;
* экранирование пользовательского вывода;
* защита от XSS;
* CSRF-защита;
* безопасные cookie;
* запрет доступа к чужим сущностям;
* пароль не возвращается через API;
* секреты не хранятся в Git;
* `.env` находится в `.gitignore`;
* в репозитории хранится `.env.example`;
* production не показывает stack trace пользователю.

## 14. Frontend

Необходимые страницы:

```text
/login
/register
/dashboard
/students
/students/new
/students/:id
/students/:id/edit
/lessons
/payments
/homeworks
/settings
```

Frontend должен поддерживать:

* загрузку данных через Fetch API;
* отображение состояния загрузки;
* отображение ошибок;
* обработку 401;
* формы создания и редактирования;
* поиск без перезагрузки страницы;
* фильтрацию;
* пагинацию;
* модальные окна;
* подтверждение опасных действий;
* уведомления об успешной операции.

## 15. AJAX-сценарии

Обязательные сценарии без полной перезагрузки страницы:

1. поиск ученика;
2. фильтрация списка учеников;
3. изменение статуса занятия;
4. создание оплаты;
5. изменение статуса домашнего задания;
6. загрузка статистики;
7. пагинация.

## 16. Дашборд

Отображать:

* количество активных учеников;
* количество занятий сегодня;
* количество занятий за текущий месяц;
* заработано за текущий месяц;
* ожидаемый доход;
* сумма задолженности;
* последние проведённые занятия;
* ближайшие занятия.

Расчёты должны выполняться на backend через SQL.

## 17. MVP

MVP считается готовым, если пользователь может:

1. зарегистрироваться;
2. войти;
3. создать ученика;
4. изменить ученика;
5. запланировать занятие;
6. изменить статус занятия;
7. добавить оплату;
8. увидеть доход за месяц;
9. найти ученика через поиск;
10. открыть приложение по публичному HTTPS-адресу.

## 18. Тесты

Минимально необходимы:

### Unit

* валидация ученика;
* расчёт статистики;
* бизнес-правила изменения статуса занятия.

### Integration

* создание ученика;
* получение только своих учеников;
* невозможность открыть чужого ученика;
* создание занятия;
* регистрация и вход.

## 19. Миграции

Не изменять структуру production-базы вручную.

Все изменения базы должны выполняться миграциями.

Каждая миграция должна иметь:

* метод применения;
* метод отката или документированное ограничение;
* понятное имя;
* отдельный коммит или понятную часть коммита.

## 20. Git-процесс

Ветки:

```text
main
develop
feature/TF-номер-краткое-название
fix/TF-номер-краткое-название
```

Примеры:

```text
feature/TF-12-create-student
feature/TF-18-lesson-status
fix/TF-27-payment-validation
```

Коммиты:

```text
feat: add student creation endpoint
fix: prevent access to another user's student
test: cover student validation
docs: update deployment guide
refactor: extract payment repository
```

Каждая задача выполняется в отдельной ветке.

Изменения попадают в `develop` через Pull Request.

Релизы попадают в `main`.

## 21. Релизы

Использовать семантическое версионирование.

Примеры:

```text
v0.1.0 — авторизация и ученики
v0.2.0 — занятия
v0.3.0 — оплаты и дашборд
v1.0.0 — публичный MVP
```

Для каждого релиза создавать GitHub Release с кратким описанием изменений.

## 22. Production

Приложение должно работать на Linux-сервере.

Production-состав:

```text
Nginx
PHP-FPM
PostgreSQL
Vue static build
Docker Compose
HTTPS
```

GitHub используется для хранения кода и CI/CD.

PHP-приложение нельзя разместить непосредственно на GitHub Pages, потому что GitHub Pages обслуживает статические файлы и не выполняет PHP.

Деплой должен быть воспроизводимым и документированным.

## 23. CI/CD

GitHub Actions должен выполнять:

* установку PHP-зависимостей;
* проверку кодстайла;
* запуск тестов;
* сборку Vue;
* проверку успешности сборки;
* после merge в `main` — развёртывание на сервер.

## 24. Документация

В репозитории должны находиться:

```text
README.md
PROJECT.md
ARCHITECTURE.md
API.md
DEPLOYMENT.md
.env.example
```

README должен содержать:

* описание проекта;
* скриншоты;
* стек;
* функциональность;
* архитектуру;
* инструкцию локального запуска;
* ссылку на production;
* тестового пользователя;
* инструкцию запуска тестов.

## 25. Правила работы Codex

Codex должен выступать как senior-разработчик и наставник, а не как исполнитель всего проекта.

Перед написанием кода Codex обязан:

1. изучить существующий проект;
2. описать затрагиваемые файлы;
3. предложить план реализации;
4. указать риски;
5. задать вопросы только при критической неоднозначности.

Codex не должен:

* переписывать проект целиком;
* менять архитектуру без объяснения;
* создавать лишние абстракции;
* добавлять библиотеку без обоснования;
* скрывать от разработчика важную логику;
* писать огромные файлы;
* смешивать SQL, HTTP и HTML в одном классе;
* выполнять несколько несвязанных задач за раз.

При реализации задачи Codex должен:

* работать только в рамках текущего тикета;
* объяснять каждое архитектурное решение;
* показывать SQL отдельно;
* сообщать о возможных уязвимостях;
* добавлять или обновлять тесты;
* не создавать код, который разработчик не сможет объяснить;
* после реализации давать список ручных проверок;
* давать вопросы для самопроверки разработчика.

Если разработчик просит обучение, Codex сначала даёт:

1. алгоритм;
2. псевдокод;
3. перечень необходимых классов и методов;
4. небольшой пример;
5. только после попытки разработчика — полный вариант.

Главная цель Codex — помочь разработчику понять и реализовать задачу, а не просто создать работающий код.
