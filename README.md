# Email Server with Docker Compose

Полнофункциональный email сервер на базе Postfix, Dovecot и MySQL.

## Характеристики

- **Postfix** - SMTP сервер для отправки и приёма писем
- **Dovecot** - IMAP сервер для доступа к письмам
- **MySQL** - база данных для хранения учётных записей пользователей
- **Открытые порты:**
  - `25` - SMTP (приём писем)
  - `465` - SMTPS (отправка писем с TLS)
  - `993` - IMAPS (доступ к письмам с TLS)
- **Хранилище писем:** файловая система (`/var/mail/vmail/`)
- **Учётные записи:** MySQL база данных
- **Без:** антивируса и спамфильтра

## Требования

- Docker и Docker Compose
- SSL сертификаты (можно создать самоподписанные)

## Установка

### 1. Создание самоподписанного сертификата

```bash
mkdir -p certs
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout certs/mail.key \
  -out certs/mail.crt \
  -subj "/CN=example.com/O=Mail Server/C=RU"
```

### 2. Настройка переменных окружения

Скопируйте `.env.example` в `.env` и отредактируйте:

```bash
cp .env.example .env
```

```env
MYSQL_ROOT_PASSWORD=your_root_password
MYSQL_DATABASE=mailserver
MYSQL_USER=mailuser
MYSQL_PASSWORD=your_password
MAIL_DOMAIN=example.com
```

### 3. Запуск сервера

```bash
docker-compose up -d
```

## Использование

### Добавление домена

```bash
docker-compose exec mysql mysql -u root -p mailserver

INSERT INTO virtual_domains (name) VALUES ('example.com');
```

### Добавление пользователя

```bash
# В MySQL
INSERT INTO virtual_users (domain_id, email, password) 
VALUES (1, 'user@example.com', SHA2('password123', 512));
```

Или через скрипт:

```bash
./add-user.sh user@example.com password123
```

### Добавление алиаса

```bash
INSERT INTO virtual_aliases (domain_id, source, destination)
VALUES (1, 'alias@example.com', 'user@example.com');
```

## Тестирование

### SMTP (порт 25)

```bash
telnet localhost 25
EHLO example.com
MAIL FROM: <sender@example.com>
RCPT TO: <user@example.com>
DATA
Subject: Test
Test message
.
QUIT
```

### IMAPS (порт 993)

Используйте клиент (Thunderbird, Outlook и т.д.):
- **Сервер:** localhost (или ваш домен)
- **Порт:** 993
- **SSL/TLS:** Да
- **Логин:** user@example.com
- **Пароль:** ваш пароль

## Логи

### Postfix
```bash
docker-compose logs -f postfix
```

### Dovecot
```bash
docker-compose logs -f dovecot
```

### MySQL
```bash
docker-compose logs -f mysql
```

## Остановка

```bash
docker-compose down
```

## Безопасность

⚠️ **Важно для production:**

1. Измените все пароли в `.env`
2. Используйте валидные SSL сертификаты (Let's Encrypt)
3. Настройте firewall правила
4. Регулярно обновляйте образы Docker
5. Используйте сильные пароли для пользователей
6. Включите fail2ban для защиты от brute-force атак

## Структура

```
.
├── docker-compose.yml
├── .env.example
├── init-db.sql
├── postfix/
│   ├── Dockerfile
│   ├── main.cf
│   ├── master.cf
│   ├── mysql-virtual-mailbox-domains.cf
│   ├── mysql-virtual-mailbox-maps.cf
│   └── mysql-virtual-alias-maps.cf
├── dovecot/
│   ├── Dockerfile
│   ├── dovecot.conf
│   └── dovecot-sql.conf.ext
└── certs/
    ├── mail.crt
    └── mail.key
```

## Проблемы

### Письма не поступают
- Проверьте правильность домена в `.env`
- Убедитесь, что домен добавлен в БД
- Проверьте логи: `docker-compose logs postfix`

### Не удаётся подключиться по IMAPS
- Проверьте сертификаты
- Убедитесь, что пользователь создан в БД
- Проверьте логи: `docker-compose logs dovecot`

### Ошибки MySQL
- Проверьте пароли в `.env`
- Убедитесь, что MySQL полностью инициализирована (может занять время)
- Проверьте логи: `docker-compose logs mysql`

## Лицензия

MIT
