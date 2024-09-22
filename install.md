# Установка софта

## Содержание
1. [Установка необходимого ПО](#установка-необходимого-по)
   - [PostgreSQL](#установка-postgresql)
   - [pgAdmin](#pgadmin)
   - [DataGrip](#datagrip)
   - [DBeaver](#dbeaver)
2. [Установка Docker](#установка-docker)
   - [Windows](#windows)
   - [Linux](#linux)
   - [macOS](#macos)
3. [Настройка PostgreSQL в Docker](#настройка-postgresql-в-docker)
   - [Запуск PostgreSQL через Docker](#запуск-postgresql-в-docker)
   - [Запуск через Docker Compose](#docker-compose)
4. [Подключение к PostgreSQL](#подключение-к-postgresql)
   - [DataGrip](#подключение-к-postgresql-через-datagrip)
   - [DBeaver](#подключение-к-postgresql-через-dbeaver)

---

## 1. Установка необходимого ПО

### Установка PostgreSQL

#### Windows
1. Перейдите на [официальный сайт PostgreSQL](https://www.postgresql.org/download/windows/) и скачайте инсталлятор для Windows.
2. Запустите установку и выберите компоненты: PostgreSQL, pgAdmin, psql (CLI).
3. Следуйте инструкциям установщика.

#### Linux
На примере Ubuntu:
1. Обновите пакеты:
   ```bash
   sudo apt update
   ```
2. Установите PostgreSQL:
   ```bash
   sudo apt install postgresql postgresql-contrib
   ```
3. Проверьте статус PostgreSQL:
   ```bash
   sudo systemctl status postgresql
   ```

#### macOS
1. Установите Homebrew, если не установлен:
   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```
2. Установите PostgreSQL:
   ```bash
   brew install postgresql
   ```
3. Запустите сервис:
   ```bash
   brew services start postgresql
   ```

### pgAdmin
**pgAdmin** — графический интерфейс для работы с PostgreSQL.

- **Windows**: Устанавливается вместе с PostgreSQL.
- **Linux**: Установка на Ubuntu:
  ```bash
  sudo apt install pgadmin4
  ```
- **macOS**: Установка через Homebrew:
  ```bash
  brew install --cask pgadmin4
  ```

### DataGrip
- Скачайте DataGrip с [официального сайта JetBrains](https://www.jetbrains.com/datagrip/).
- Поддерживает работу с PostgreSQL, автодополнение, визуализацию схем.

### DBeaver
- Скачайте DBeaver с [официального сайта](https://dbeaver.io/download/).
- Поддерживает PostgreSQL, бесплатен и кроссплатформен.

---

## 2. Установка Docker

### Windows
1. Скачайте **Docker Desktop** с [официального сайта](https://www.docker.com/products/docker-desktop).
2. Установите, следуя инструкциям.
3. Перезагрузите компьютер и запустите Docker Desktop.

### Linux

#### Ubuntu:
1. Установите зависимости:
   ```bash
   sudo apt update
   sudo apt install apt-transport-https ca-certificates curl software-properties-common
   ```
2. Добавьте ключ репозитория Docker и установите репозиторий:
   ```bash
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
   ```
3. Установите Docker:
   ```bash
   sudo apt update
   sudo apt install docker-ce
   ```
4. Убедитесь, что Docker запущен:
   ```bash
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

5. Добавьте текущего пользователя в группу Docker (чтобы не использовать `sudo`):
   ```bash
   sudo usermod -aG docker ${USER}
   ```
   Перезагрузите систему, чтобы изменения вступили в силу.

6. Проверьте установку:
   ```bash
   docker --version
   ```

#### Fedora:
1. Установите Docker:
   ```bash
   sudo dnf install docker
   ```
2. Запустите сервис Docker:
   ```bash
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

### macOS
1. Скачайте **Docker Desktop** с [официального сайта](https://www.docker.com/products/docker-desktop).
2. Установите и запустите Docker Desktop.

---

## 3. Настройка PostgreSQL в Docker

### Запуск PostgreSQL в Docker
1. Для запуска контейнера PostgreSQL используйте команду:
   ```bash
   docker run --name my-postgres -p 5432:5432 -e POSTGRES_USER=myuser -e POSTGRES_PASSWORD=mypassword -e POSTGRES_DB=mydb -d postgres:latest
   ```
   **Пояснение:**
   - `--name my-postgres`: имя контейнера.
   - `-p 5432:5432`: маппинг портов.
   - `-e POSTGRES_USER=myuser`: пользователь PostgreSQL.
   - `-e POSTGRES_PASSWORD=mypassword`: пароль.
   - `-e POSTGRES_DB=mydb`: база данных.
   - `-d postgres:latest`: запуск в фоне с последней версией PostgreSQL.

2. Проверьте, что контейнер запущен:
   ```bash
   docker ps
   ```

3. Для подключения к контейнеру используйте:
   ```bash
   docker exec -it my-postgres psql -U myuser -d mydb
   ```

### Docker Compose
1. Создайте файл `docker-compose.yml` с содержимым:

   ```yaml
   version: "3.9"
   services:
     postgres:
       image: postgres:latest
       environment:
         POSTGRES_DB: "mydb"
         POSTGRES_USER: "myuser"
         POSTGRES_PASSWORD: "mypassword"
       ports:
         - "5432:5432"
       volumes:
         - ./pgdata:/var/lib/postgresql/data
   ```

2. Запустите PostgreSQL с помощью Docker Compose:
   ```bash
   docker-compose up -d
   ```

3. Остановите контейнер:
   ```bash
   docker-compose down
   ```

---

## 4. Подключение к PostgreSQL

### Подключение к PostgreSQL через DataGrip
1. Откройте DataGrip: **File** → **New** → **Data Source** → **PostgreSQL**.
2. Введите следующие данные:
   - **Host**: `localhost`
   - **Port**: `5432`
   - **Database**: `mydb`
   - **User**: `myuser`
   - **Password**: `mypassword`
3. Нажмите **Test Connection** для проверки соединения.

### Подключение к PostgreSQL через DBeaver
1. Запустите DBeaver и выберите **Database** → **New Database Connection**.
2. Выберите **PostgreSQL** из списка.
3. Введите следующие параметры:
   - **Host**: `localhost`
   - **Port**: `5432`
   - **Database**: `mydb`
   - **Username**: `myuser`
   - **Password**: `mypassword`
4. Нажмите **Test Connection** для проверки соединения.
5. Если соединение успешно, нажмите **Finish**.
