# Домашнее задание к занятию «Репликация и масштабирование. Часть 1» - `Чеботников М.Б.`

### Инструкция по выполнению домашнего задания

1. Сделайте fork [репозитория c шаблоном решения](https://github.com/netology-code/sys-pattern-homework) к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/gitlab-hw или https://github.com/имя-вашего-репозитория/8-03-hw).
2. Выполните клонирование этого репозитория к себе на ПК с помощью команды `git clone`.
3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
   - впишите вверху название занятия и ваши фамилию и имя;
   - в каждом задании добавьте решение в требуемом виде: текст/код/скриншоты/ссылка;
   - для корректного добавления скриншотов воспользуйтесь инструкцией [«Как вставить скриншот в шаблон с решением»](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md);
   - при оформлении используйте возможности языка разметки md. Коротко об этом можно посмотреть в [инструкции по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md).
4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`).
5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
6. Любые вопросы задавайте в чате учебной группы и/или в разделе «Вопросы по заданию» в личном кабинете.

---

### Задание 1
На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.

Ответить в свободной форме.

#### ОТВЕТ:
В режиме "master-slave" операции с данными (запись, чтение, обновление, удаление) производятся на "master", "slave" получает данные от "master", а для остальных и работает только на чтение.

В режим "master-master" операции с данными могут производиться на всех серверах.

---

### Задание 2
Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.

Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.

#### ОТВЕТ:
`docker-compose.yml`
```
services:
  mysql-master:
    image: mysql:8.0
    container_name: mysql-master
    command:
      - --server-id=1
      - --log-bin=mysql-bin
      - --binlog-format=ROW
      - --bind-address=0.0.0.0
      - --skip-name-resolve
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
    ports:
      - "3306:3306"
    volumes:
      - master_mysql_data:/var/lib/mysql
    networks:
      - repl-net

  mysql-slave:
    image: mysql:8.0
    container_name: mysql-slave
    command:
      - --server-id=2
      - --relay-log=mysql-relay-bin
      - --read-only=1
      - --bind-address=0.0.0.0
      - --skip-name-resolve
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
    ports:
      - "33066:3306"
    volumes:
      - slave_mysql_data:/var/lib/mysql
    networks:
      - repl-net
    depends_on:
      - mysql-master

volumes:
  master_mysql_data:
  slave_mysql_data:

networks:
  repl-net:
    driver: bridge
```

<img width="297" height="152" alt="1" src="https://github.com/user-attachments/assets/207641dc-3492-43fe-aa04-c312de0323cd" />


<img width="560" height="91" alt="2" src="https://github.com/user-attachments/assets/366ad564-9e6c-4655-9686-d4788554107a" />


`На master выполняем код: `
```
docker exec -it mysql-master mysql -uroot -prootpass -e "
CREATE DATABASE IF NOT EXISTS \`home-db-12-6\`;
USE \`home-db-12-6\`;
CREATE TABLE employees (id INT PRIMARY KEY, name VARCHAR(50));
INSERT INTO employees VALUES (1, 'Test1'), (2, 'Test2');
SELECT * FROM employees;
"
```

<img width="563" height="202" alt="3" src="https://github.com/user-attachments/assets/77e9daaa-157c-4a86-a30e-bbc3a1d8a449" />


`На slave выполняем код: `
```
docker exec -it mysql-slave mysql -uroot -prootpass -e "SELECT * FROM \`home-db-12-6\`.employees;"
```

<img width="561" height="118" alt="4" src="https://github.com/user-attachments/assets/1c5e7677-96d7-4e66-85b8-b6f31be64890" />


---




