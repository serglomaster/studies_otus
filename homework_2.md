Домашнее задание №2

Работа с уровнями изоляции транзакции в PostgreSQL.

Работа выполнена в Yandex Cloud.

Для внешнего доступа к виртуальной машине использовалась программа PuTTy.

Создано облако cloud-workcul. Добавлен пользователь
[ifti@yandex.ru](mailto:ifti@yandex.ru).

В каталоге testpg создана виртуальная машина virtual1.

Программой puttygen сгенерированы публичный и приватный ключи ssh для доступа к
postgresql серверу на виртуальной машине.

На виртуальной машине установлен postgresql сервер:

sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb
http://apt.postgresql.org/pub/repos/apt \$(lsb_release -cs)-pgdg main" \>
/etc/apt/sources.list.d/pgdg.list' && wget --quiet -O -
https://www.postgresql.org/media/keys/ACCC4CF8.asc \| sudo apt-key add - && sudo
apt-get update && sudo apt-get -y install postgresql

Для внешнего доступа к серверу отредактированы файлы:

postgresql.conf параметр listen_addresses = '\*' разрешен доступ со всех IP
адресов

pg_hba.conf добавлена строка

host all all 0.0.0.0/0 md5

доступ со всех ip адресов по паролю

пароль пользователя postgres : 12345678

Создана база test.

Запущены две сессии, удаленного подключения к виртуальной машине по SSH.

В обеих сессиях подключились к серверу пользователем postgres

В обеих сессиях выключен autocommit

read commited транзакции:

в первой сессии создаем новую таблицу с данными

create table persons(id serial, first_name text, second_name text);

insert into persons(first_name, second_name) values('ivan', 'ivanov');

insert into persons(first_name, second_name) values('petr', 'petrov');

commit;

в первой сессии добавляем новую запись:

insert into persons(first_name, second_name) values('sergey', 'sergeev');

Во второй сессии выполним запрос:

select \* from persons;

Новая запись из первой сессии не видна так как там не закрыта транзакция.

После выполнения commit в первой сессии запрос select во второй сессии вернет
все записи, включая новую.

repeatable read транзации в обеих сессиях:

в первой сессии добавляем новую запись:

insert into persons(first_name, second_name) values('sveta', 'svetova');

Во второй сессии выполним запрос:

select \* from persons;

Новая запись из первой сессии не видна так как там не закрыта транзакция.

Закроем транзакцию в первой сессии: commit

По запросу из второй сессии select новая запись не видна так как не закрыта
транзакция во второй сессии.

После закрытия транзакции во второй сессии будет видна новая запись и во второй
сесии.
