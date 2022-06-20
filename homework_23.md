ДЗ 23
-- Триггерная функция добавляет, изменяет данные в таблице отчета good_sum_mart
-- При изменении данных в таблице sales
CREATE OR REPLACE FUNCTION fun_sales_trg()
RETURNS trigger
AS
$TRIG_FUNC$
declare
   _good_name   varchar(63);
BEGIN

    if (TG_OP='INSERT' or TG_OP='UPDATE') then  -- при добавлении изменении используем new
        select good_name from goods where goods_id=new.good_id into _good_name; -- по id определяем имя товара
        delete from good_sum_mart where good_name=_good_name;  -- удаляем старую запись 
      --  новая запись с новой суммой продаж
        insert into good_sum_mart (good_name, sum_sale) SELECT G.good_name, sum(G.good_price * S.sales_qty)
          FROM goods G
          INNER JOIN sales S ON S.good_id = G.goods_id where S.good_id=new.good_id
          GROUP BY G.good_name;
	RETURN NEW;
    end if;
    if (TG_OP='DELETE') then  -- при удалении используем запись old
        select good_name from goods where goods_id=old.good_id into _good_name; -- по id определяем имя товара
        delete from good_sum_mart where good_name=_good_name;  -- удаляем старую запись 
      --  новая запись с новой суммой продаж
        insert into good_sum_mart (good_name, sum_sale) SELECT G.good_name, sum(G.good_price * S.sales_qty)
          FROM goods G
          INNER JOIN sales S ON S.good_id = G.goods_id where S.good_id=old.good_id
          GROUP BY G.good_name;
	RETURN OLD;
    end if;
END;
$TRIG_FUNC$
  LANGUAGE plpgsql;

-- собственно триггер 
CREATE TRIGGER tr_sales
AFTER INSERT OR UPDATE OR DELETE
ON sales
FOR EACH ROW
EXECUTE PROCEDURE fun_sales_trg();
