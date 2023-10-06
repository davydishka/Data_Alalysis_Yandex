# SQL основы
## Описание проекта
 Необходимо проанализировать данные о фондах и инвестициях и написать запросы к базе. 

### 1. Сколько компаний закрылось.

---
SELECT COUNT(status)\
FROM company\
WHERE status = 'closed';

### 2. Отобразить количество привлечённых средств для новостных компаний США. 

---
SELECT funding_total \
FROM company\
WHERE category_code='news'\
AND country_code = 'USA'\
ORDER BY funding_total DESC;

### 3. Найти общую сумму сделок по покупке одних компаний другими в долларах. 
### Отобрать сделки, которые осуществлялись только за наличные с 2011 по 2013 год включительно.

---
SELECT SUM(price_amount)\
FROM acquisition\
WHERE term_code = 'cash'\
AND CAST(acquired_at AS date) BETWEEN '2011-01-01' AND '2013-12-31';

### 4. Отобразите имя, фамилию и названия аккаунтов людей в твиттере, 
### у которых названия аккаунтов начинаются на 'Silver'.

---
SELECT first_name,\
    last_name,\
    twitter_username\
FROM people\
WHERE twitter_username LIKE 'Silver%';

### 5. Вывести на экран всю информацию о людях, у которых названия аккаунтов в твиттере 
### содержат подстроку 'money', а фамилия начинается на 'K'.

---
SELECT *\
FROM people\
WHERE twitter_username LIKE '%money%' \
AND last_name LIKE 'K%';

### 6. Для каждой страны отобразить общую сумму привлечённых инвестиций, которые получили компании, 
зарегистрированные в этой стране. \
Страну, в которой зарегистрирована компания, можно определить по коду страны. \
Отсортировать данные по убыванию суммы.

---
SELECT country_code,\
----SUM(funding_total)\
FROM company\
GROUP BY country_code\
ORDER BY SUM(funding_total) DESC;

### 7. Составить таблицу, в которую войдёт дата проведения раунда, 
а также минимальное и максимальное значения суммы инвестиций, привлечённых в эту дату.\
Оставить в итоговой таблице только те записи, в которых минимальное значение суммы инвестиций \
не равно нулю и не равно максимальному значению.

---
SELECT funded_at,\
    MIN(raised_amount),\
    MAX(raised_amount)\
FROM funding_round\
GROUP BY funded_at\
HAVING MIN(raised_amount) != 0 \
AND MIN(raised_amount) != MAX(raised_amount);

### 8. Создать поле с категориями:
Для фондов, которые инвестируют в 100 и более компаний, назначить категорию high_activity.\
Для фондов, которые инвестируют в 20 и более компаний до 100, назначить категорию middle_activity.\
Если количество инвестируемых компаний фонда не достигает 20, назначить категорию low_activity.\
Отобразить все поля таблицы fund и новое поле с категориями.

---
SELECT *,\
CASE\
    WHEN invested_companies >= 100 THEN 'high_activity'\
    WHEN invested_companies >= 20 AND invested_companies < 100 THEN 'middle_activity'\
    WHEN invested_companies < 20 THEN 'low_activity'\
END\
FROM fund;

### 9. Для каждой из категорий, назначенных в предыдущем задании, посчитать округлённое 
до ближайшего целого числа среднее количество инвестиционных раундов, в которых фонд принимал участие. \
Вывести на экран категории и среднее число инвестиционных раундов. \
Отсортировать таблицу по возрастанию среднего.

---
SELECT \
       CASE\
           WHEN invested_companies>=100 THEN 'high_activity'\
           WHEN invested_companies>=20 THEN 'middle_activity'\
           ELSE 'low_activity'\
       END AS activity,\
       ROUND(AVG(investment_rounds)) AS avg_rounds\
FROM fund\
GROUP BY CASE\
           WHEN invested_companies>=100 THEN 'high_activity'\
           WHEN invested_companies>=20 THEN 'middle_activity'\
           ELSE 'low_activity'\
       END\
ORDER BY avg_rounds;

### 10. Проанализировать, в каких странах находятся фонды, которые чаще всего инвестируют в стартапы. 
Для каждой страны посчитать минимальное, максимальное и среднее число компаний, \
в которые инвестировали фонды этой страны, основанные с 2010 по 2012 год включительно. \
Исключить страны с фондами, у которых минимальное число компаний, получивших инвестиции, равно нулю. \
Выгрузить 10 самых активных стран-инвесторов: отсортировать таблицу по среднему количеству компаний от большего к меньшему.\ 
Затем добавить сортировку по коду страны в лексикографическом порядке.

---
SELECT country_code,\
    MIN(invested_companies),\
    MAX(invested_companies),\
    AVG(invested_companies)\
FROM fund\
WHERE CAST(founded_at AS date) BETWEEN '2010-01-01' AND '2012-12-31'\
GROUP BY country_code\
HAVING MIN(invested_companies) > 0\
ORDER BY AVG(invested_companies) DESC, country_code\
LIMIT 10; 

### 11. Отобразить имя и фамилию всех сотрудников стартапов. 
Добавить поле с названием учебного заведения, которое окончил сотрудник, если эта информация известна.

---
SELECT p.first_name,\
    p.last_name,\
    e.instituition\
FROM people AS p\
LEFT JOIN education AS e ON p.id=e.person_id;

### 12. Для каждой компании найти количество учебных заведений, которые окончили её сотрудники. 
Вывести название компании и число уникальных названий учебных заведений. \
Составить топ-5 компаний по количеству университетов.

---
SELECT c.name,\
    COUNT(DISTINCT e.instituition) AS count_univ\
FROM company AS c\
JOIN people AS p ON c.id=p.company_id\
JOIN education AS e ON p.id=e.person_id\
GROUP BY c.name\
ORDER BY count_univ DESC\
LIMIT 5;

### 13. Составить список с уникальными названиями закрытых компаний, 
для которых первый раунд финансирования оказался последним.

---
SELECT DISTINCT c.name\
FROM company AS c\
LEFT JOIN funding_round AS f ON c.id=f.company_id\
WHERE is_first_round = 1\
AND is_last_round = 1\
AND c.status = 'closed'\
GROUP BY c.name;

### 14. Составить список уникальных номеров сотрудников, которые работают в компаниях, отобранных в предыдущем задании.

---
SELECT id\
FROM people\
WHERE company_id IN (SELECT c.id\
FROM company AS c\
LEFT JOIN funding_round AS f ON c.id=f.company_id\
WHERE is_first_round = 1\
AND is_last_round = 1\
AND c.status = 'closed'\
GROUP BY c.id);

### 15. Составить таблицу, куда войдут уникальные пары с номерами сотрудников 
из предыдущей задачи и учебным заведением, которое окончил сотрудник.

---
SELECT DISTINCT p.id,\
    e.instituition\
FROM people AS p\
JOIN education AS e ON e.person_id=p.id\
WHERE company_id IN (SELECT c.id\
FROM company AS c\
LEFT JOIN funding_round AS f ON c.id=f.company_id\
WHERE is_first_round = 1\
AND is_last_round = 1\
AND c.status = 'closed'\
GROUP BY c.id);

### 16. Посчитать количество учебных заведений для каждого сотрудника из предыдущего задания. 
При подсчёте учесть, что некоторые сотрудники могли окончить одно и то же заведение дважды.

---
SELECT DISTINCT p.id,\
    COUNT(e.instituition)\
FROM people AS p\
JOIN education AS e ON e.person_id=p.id\
WHERE company_id IN \
(SELECT c.id\
FROM company AS c\
LEFT JOIN funding_round AS f ON c.id=f.company_id\
WHERE is_first_round = 1\
AND is_last_round = 1\
AND c.status = 'closed'\
GROUP BY c.id)\
GROUP BY p.id;

### 17. Дополнить предыдущий запрос и вывести среднее число учебных заведений (всех, не только уникальных), 
которые окончили сотрудники разных компаний. Нужно вывести только одну запись, группировка здесь не понадобится.

---
WITH\
s AS \
(SELECT DISTINCT p.id,\
    COUNT(e.instituition) AS inst\
FROM people AS p\
JOIN education AS e ON e.person_id=p.id\
WHERE company_id IN \
(SELECT c.id\
FROM company AS c\
LEFT JOIN funding_round AS f ON c.id=f.company_id\
WHERE is_first_round = 1\
AND is_last_round = 1\
AND c.status = 'closed'\
GROUP BY c.id)\
GROUP BY p.id)\

SELECT AVG(inst) \
FROM s;

### 18. Написать похожий запрос: вывести среднее число учебных заведений (всех, не только уникальных), 
которые окончили сотрудники Facebook*.

---
WITH\
s AS \
(SELECT DISTINCT p.id,\
    COUNT(e.instituition) AS inst\
FROM people AS p\
JOIN education AS e ON e.person_id=p.id\
WHERE company_id IN \
(SELECT c.id\
FROM company AS c\
WHERE \
 c.name = 'Facebook'\
GROUP BY c.id)\
GROUP BY p.id)\

SELECT AVG(inst) \
FROM s;

### 19. Составить таблицу из полей:
- name_of_fund — название фонда;\
- name_of_company — название компании;\
- amount — сумма инвестиций, которую привлекла компания в раунде.\
В таблицу войдут данные о компаниях, в истории которых было больше шести важных этапов, \
а раунды финансирования проходили с 2012 по 2013 год включительно.

---
SELECT f.name AS name_of_fund,\
    c.name AS name_of_company,\
    fr.raised_amount AS amount\
FROM investment AS i\
LEFT JOIN company AS c ON c.id=i.company_id\
LEFT JOIN fund AS f ON i.fund_id= f.id\
LEFT JOIN funding_round AS fr ON i.funding_round_id=fr.id\
WHERE c.milestones > 6\
AND CAST(fr.funded_at AS date) BETWEEN '2012-01-01' AND '2013-12-31';\

### 20. Выгрузить таблицу, в которой будут такие поля:
- название компании-покупателя;
- сумма сделки;
- название компании, которую купили;
- сумма инвестиций, вложенных в купленную компанию;
- доля, которая отображает, во сколько раз сумма покупки превысила сумму вложенных \
в компанию инвестиций, округлённая до ближайшего целого числа.\
Не учитывать те сделки, в которых сумма покупки равна нулю. Если сумма инвестиций в компанию равна нулю, \
исключить такую компанию из таблицы. \
Отсортировать таблицу по сумме сделки от большей к меньшей, а затем по названию купленной компании \
в лексикографическом порядке. Ограничить таблицу первыми десятью записями.

---
SELECT c1.name AS buyer,\
    a.price_amount AS sum_deal,\
    c2.name AS startup,\
    c2.funding_total AS sum_invest,\
   -- ROUND(sum_deal/sum_invest) AS share\
    ROUND(a.price_amount/c2.funding_total) AS share\
FROM acquisition AS a\
INNER JOIN company AS c1 ON a.acquiring_company_id=c1.id\
INNER JOIN company AS c2 ON a.acquired_company_id=c2.id\
WHERE price_amount != 0\
AND c2.funding_total != 0\
ORDER BY sum_deal DESC, startup\
LIMIT 10;

### 21. Выгрузить таблицу, в которую войдут названия компаний из категории social, 
получившие финансирование с 2010 по 2013 год включительно. \
Проверить, что сумма инвестиций не равна нулю. Вывести также номер месяца, в котором проходил раунд финансирования.

---
SELECT c.name,\
    EXTRACT(MONTH FROM CAST(f.funded_at AS date))\
FROM company AS c\
LEFT JOIN funding_round AS f ON c.id=f.company_id\
WHERE c.category_code = 'social'\
AND\
CAST(f.funded_at AS date) BETWEEN '2010-01-01' AND '2013-12-31'\
AND f.raised_amount !=0;

### 22. Отбрать данные по месяцам с 2010 по 2013 год, когда проходили инвестиционные раунды. 
Сгруппировать данные по номеру месяца и получить таблицу, в которой будут поля:
- номер месяца, в котором проходили раунды;
- количество уникальных названий фондов из США, которые инвестировали в этом месяце;
- количество компаний, купленных за этот месяц;
- общая сумма сделок по покупкам в этом месяце.

---
WITH\
c1 AS\
(SELECT EXTRACT(MONTH FROM CAST(fr.funded_at AS date)) AS month,\
COUNT(DISTINCT f.name) AS fonds\

FROM funding_round AS fr\
LEFT JOIN investment AS i ON i.funding_round_id = fr.id\
LEFT JOIN fund AS f ON f.id = i.fund_id\

WHERE CAST(fr.funded_at AS date) BETWEEN '2010-01-01' AND '2013-12-31'\
AND f.country_code = 'USA'\
GROUP BY month),\

c2 AS (  SELECT EXTRACT(MONTH FROM acquired_at) AS month,\
    COUNT(acquired_company_id) AS companies,\
    SUM(price_amount) AS price\
    FROM acquisition\
        WHERE acquired_at BETWEEN '2010-01-01' AND '2013-12-31'\
    GROUP BY month)\
    
SELECT c1.month,\
c1.fonds,\
c2.companies,\
c2.price\
FROM c1 FULL JOIN c2 ON c1.month=c2.month


### 23. Составить сводную таблицу и вывести среднюю сумму инвестиций для стран, 
в которых есть стартапы, зарегистрированные в 2011, 2012 и 2013 годах. \
Данные за каждый год должны быть в отдельном поле. \
Отсортировать таблицу по среднему значению инвестиций за 2011 год от большего к меньшему.

---
WITH\
i2011 AS (SELECT country_code,\
    AVG(funding_total) AS sum_2011\
FROM company\
WHERE EXTRACT(YEAR FROM CAST(founded_at AS date)) = 2011\
GROUP BY country_code),\

i2012 AS (SELECT country_code,\
    AVG(funding_total) AS sum_2012\
FROM company\
WHERE EXTRACT(YEAR FROM CAST(founded_at AS date)) = 2012\
GROUP BY country_code),\

i2013 AS (SELECT country_code,\
    AVG(funding_total) AS sum_2013\
FROM company\
WHERE EXTRACT(YEAR FROM CAST(founded_at AS date)) = 2013\
GROUP BY country_code)\

SELECT \
i2011.country_code,\
  sum_2011,\
    sum_2012,\
    sum_2013\
FROM i2011\
JOIN i2012 ON  i2011.country_code=i2012.country_code\
JOIN i2013 ON  i2011.country_code=i2013.country_code\
ORDER BY sum_2011 DESC;
