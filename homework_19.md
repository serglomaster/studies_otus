1 вариант:
Есть тестовая таблица Persons с полями:
id serial, user_name text, address text, comment text, City text
В ней 10000 записей.

Создадим индекс на поле user_name
CREATE INDEX idx_name ON persons (user_name);
Запрос который использует данный индекс:
select * from persons where user_name='Smit';
план запроса:
Index Scan using idx_nm on test  (cost=0.29..8.30 rows=1 width=25)
  Index Cond: (user_name='Smit'::text)

Для полнотекстового поиска по полю address создадим индекс:
CREATE INDEX idx_gin_address ON persons USING gin (to_tsvector('english', address));
Так как PostgreSQL для полнотекстового поиска использует специальный формат tsvector то
в индексе используется функция, преобразующая текст к этому формату.

Создадим индекс на часть таблицы
Примем условие, что будем работать с первыми 1000 записями  те id<=1000
CREATE INDEX ind_city_m ON persons (id)  WHERE id<=1000;
что бы запрос использовал этот индекс нужно в условие where вводить ограничение на id до 1000
например
select * from persons where id<50 and address like 'Mo%';

Индекс на два поля: user_name и city
CREATE INDEX idx_usercity ON persons (user_name, city);

Для использования этого индекса в запросе важно какие поля в условии отбора указываются.
При условии по двум полям или по первому полю user_name индекс будет использоваться.
При условии по полю city индекс не будет использоваться.
