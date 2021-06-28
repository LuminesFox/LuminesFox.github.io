# Инструкция по развертке


## Установка необходимых пакетов

```sh
yum install python3.9 -y
yum install nano  -y # по необходимости

dnf install nginx -y
dnf install git-all -y

# Remi for Redis
dnf install epel-release -y
dnf install http://rpms.remirepo.net/enterprise/remi-release-8.rpm -y
```

## Настройка `git`

### Настраиваем аккаунт для `git`:
    ```sh
    git config --global user.name "<your username>"
    git config --global user.email "<your email>"
    ```

### Генерируем `ssh` ключ:
  ```sh
  ssh-keygen -t rsa -b 4096 -C "<your email>"
  eval "$(ssh-agent -s)"
  cat /root/.ssh/id_rsa.pub
  ```
	
  > На выходе получаем ключ который надо сохранить в настройках на сайте гита
  > 
	> Далее клонируем репозиторий на сервер в папку /www папку надо создать
	> 
	> (путь до корня проекта /www/<project_name>)



## Установка Postgresql

> Для CentOS 7: https://www.techsupportpk.com/2020/12/how-to-install-postgresql-release-13-centos.html

```sh
dnf module list postgresql  # выведет список доступных версий PostgreSQL
dnf install @postgresql:13 # говорим что нам нужна 13 версия из списка
postgresql-setup initdb
systemctl enable --now postgresql

sudo nano /var/lib/pgsql/data/pg_hba.conf  # `ident` заменить на password как показано на скриншоте в крайнем правом столбце)
```

Заходим под постгреса и ставим пароль на роль `postgres`:
  ```sh
	su - postgres  # sudo -u postgres
	psql

	ALTER USER postgres WITH ENCRYPTED PASSWORD '333';
	ALTER USER postgres WITH PASSWORD '333';
	CREATE DATABASE "<project_name>"
  ```
  
  ```sh
	systemctl restart postgresql.service
  ```
 
## Настройка `NGINX`

### Добавим фаервол

```sh
systemctl status firewalld
systemctl start firewalld
systemctl restart firewalld
systemctl enable firewalld
firewall-cmd --permanent --add-service=http #>success
firewall-cmd --permanent --list-all #Проверка того что фаервол работает
sudo firewall-cmd --reload #>success
```

### Включаем `NGINX`

```sh
systemctl enable nginx
systemctl start nginx
```

## Настройка [Redis](https://dataenginer.ru/?p=6998)

```sh
dnf config-manager --enable remi
dnf repolist
```

> Для CentOS 7: https://baks.dev/article/centos/how-to-install-and-configure-redis-on-centos-7


## Настройка проекта

Все действия производятся в директории проекта

### Backend

Действия производятся в папке `backend`

#### Настройка `Python Virtual Env`

```sh
python3.9 -m venv env
source ./env/bin/activate
pip install -r requirements.txt
```

#### Миграции

```sh
./migrate.py migrate all
```

#### Создание `.env` файла

```sh
cp .env.sample .env
nano .env
```

Необходимо заполнить недостоющие поля в `.env` файле

### Frontend

#### Сборка

```sh
npm run build
```

#### Запуск

```sh
npx serve -l 8080 -n -s dist
```
