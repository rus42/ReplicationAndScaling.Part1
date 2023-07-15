Репликация и масштабирование. Часть 1. Васёв А.В.

## Задание 1
### На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.

Master-Slave репликация

При этом подходе выделяется один основной сервер БД, который называется Master. На нем происходят все изменения данных (запросы SQL INSERT/UPDATE/DELETE). Slave сервер постоянно копирует все изменения с Master. На Slave сервер отправляются запросы чтения данных (запросы SQL SELECT). Таким образом Master сервер отвечает за изменения данных, а Slave за чтение.

Master-Master репикация

При этом подходе любой из серверов может использоваться как для чтения, так и для записи. В случаче вызода из строя одного из серверов, с большей вероятностью потери данных будут и на других Master-серверах. Последующее восстановление также сильно затрудняется необходимостью ручного анализа данных, которые успели либо не успели скопироваться. Обычно данная схема используется, когда на всех серверах где нужно обеспечить и запись, и чтение информации. Использовать асинхронный Master-Master для одновременной записи в обе БД без знания подводных камней — опасно и ненадежно и применяется в редких случаях.

## Задание 2
### Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.

Этапы настройки серверов:
```java
1. для обеих виртуальных машин
	$ mkdir -p /var/log/mysql // создание каталогов для хранения логов
2. для обеих виртуальных машин
	$ mysqld --initialize // инициализация директории данных MySQL
3. для обеих виртуальных машин
	$ chown -R mysql: /var/lib/mysql // изменение владельца каталога
	$ chown -R mysql: /var/log/mysql // изменение владельца каталога
4. для обеих виртуальных машин
	$ nano /etc/my.cnf
	4.1. виртуальная машина mysql1
		bind-address=0.0.0.0
		server-id=1
		log_bin=/var/log/mysql/mybin.log
	4.2. виртуальная машина mysql2
		bind-address=0.0.0.0
		server-id=2
		log_bin=/var/log/mysql/mybin.log
5. для обеих виртуальных машин 
	cat /var/log/mysqld.log - получение временного пароля
6. для обеих виртуальных машин 
	$ systemctl start mysqld // запуску службы mysqld
7. для обеих виртуальных машин 
	$ mysql -p // авторизация с временным паролем
8. для обеих виртуальных машин
	mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'Passw0rd'; // изменение пароля root
	mysql> FLUSH PRIVILEGES; перепрочтение таблицы предоставления привилегий сервером
9. для обеих виртуальных машин
	mysql> CREATE USER 'repl'@'%' IDENTIFIED WITH mysql_native_password BY 'Passw0rdRepl; // создание пользователя, от имени которого будет выполняться репликация
10. для обеих виртуальных машин
	mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%'; // пердоставление привелегий на репликацию
11. виртуальная машина mysql1
	mysql> show master status;
11. виртуальная машина mysql2
	mysql> CHANGE MASTER TO MASTER_HOST='10.129.0.13', MASTER_USER='repl', MASTER_PASSWORD='Passw0rdRepl, MASTER_LOG_FILE = 'mybin.000001', MASTER_LOG_POS = 1161; // настройка master - slave репликации
	mysql> START SLAVE;
	mysql> SHOW SLAVE STATUS\G;
```

![alt text](https://github.com/rus42/ReplicationAndScaling.Part1/blob/main/Task_2.png)

## Задание 3
### Выполните конфигурацию master-master репликации. Произведите проверку.

Этапы ДОнастройки серверов:

```java
1. виртуальная машина mysql2
	mysql> show master status;
2. виртуальная машина mysql2
	mysql> CHANGE MASTER TO MASTER_HOST='10.129.0.26', MASTER_USER='repl', MASTER_PASSWORD='Passw0rdRepl, MASTER_LOG_FILE = 'mybin.000001', MASTER_LOG_POS = 1366; // настройка master - master репликации
	mysql> START SLAVE;
```



