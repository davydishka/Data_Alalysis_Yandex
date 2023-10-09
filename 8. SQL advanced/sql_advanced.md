# SQL advanced

## Описание проекта
  Работаем с базой данных StackOverflow — сервиса вопросов и ответов о программировании. 
База содержит данные о постах за 2008 год.

## Описание данных:
*stackoverflow.badges* - Хранит информацию о значках, которые присуждаются за разные достижения. 
- id	Идентификатор значка, первичный ключ таблицы
- name	Название значка
- user_id	Идентификатор пользователя, которому присвоили значок, внешний ключ, отсылающий к таблице users
- creation_date	Дата присвоения значка

*stackoverflow.post_types* - Содержит информацию о типе постов. Их может быть два:
- Question — пост с вопросом;
- Answer — пост с ответом.

- id	Идентификатор поста, первичный ключ таблицы
- type	Тип поста

*stackoverflow.posts* - Содержит информацию о постах.
- id	Идентификатор поста, первичный ключ таблицы
- title	Заголовок поста
- creation_date	Дата создания поста
- favorites_count	Число, которое показывает, сколько раз пост добавили в «Закладки»
- last_activity_date	Дата последнего действия в посте, например комментария
- last_edit_date	Дата последнего изменения поста
- user_id	Идентификатор пользователя, который создал пост, внешний ключ к таблице users
- parent_id	Если пост написали в ответ на другую публикацию, в это поле попадёт идентификатор поста с вопросом
- post_type_id	Идентификатор типа поста, внешний ключ к таблице post_types
- score	Количество очков, которое набрал пост
- views_count	Количество просмотров

*stackoverflow.users* - Содержит информацию о пользователях.
- id	Идентификатор пользователя, первичный ключ таблицы
- creation_date	Дата регистрации пользователя
- display_name	Имя пользователя
- last_access_date	Дата последнего входа
- location	Местоположение
- reputation	Очки репутации, которые получают за хорошие вопросы и полезные ответы
- views	Число просмотров профиля пользователя

*stackoverflow.vote_types* - Содержит информацию о типах голосов. 
- Голос — это метка, которую пользователи ставят посту. Типов бывает несколько: 
- UpMod — такую отметку получают посты с вопросами или ответами, которые пользователи посчитали уместными и полезными.
- DownMod — такую отметку получают посты, которые показались пользователям наименее полезными.
- Close — такую метку ставят опытные пользователи сервиса, если заданный вопрос нужно доработать или он вообще не подходит для платформы.
- Offensive — такую метку могут поставить, если пользователь ответил на вопрос в грубой и оскорбительной манере, например, указав на неопытность автора поста.
- Spam — такую метку ставят в случае, если пост пользователя выглядит откровенной рекламой.

- id	Идентификатор типа голоса, первичный ключ
- name	Название метки

*stackoverflow.votes* - Содержит информацию о голосах за посты. 
- id	Идентификатор голоса, первичный ключ
- post_id	Идентификатор поста, внешний ключ к таблице posts
- user_id	Идентификатор пользователя, который поставил посту голос, внешний ключ к таблице users
- bounty_amount	Сумма вознаграждения, которое назначают, чтобы привлечь внимание к посту
- vote_type_id	Идентификатор типа голоса, внешний ключ к таблице vote_types
- creation_date	Дата назначения голоса



## 1. Найдите количество вопросов, которые набрали больше 300 очков или как минимум 100 раз были добавлены в «Закладки».

---
SELECT COUNT(p.post_type_id)\
FROM stackoverflow.posts p\
LEFT JOIN stackoverflow.post_types t ON p.post_type_id=t.id\
WHERE \
(type = 'Question' AND score > 300)\
OR (type = 'Question' AND favorites_count >= 100);

## 2. Сколько в среднем в день задавали вопросов с 1 по 18 ноября 2008 включительно? Результат округлите до целого числа.

---
SELECT ROUND(AVG(quest_count))\
FROM \
(\
    SELECT DISTINCT creation_date::date,\
 COUNT(*) OVER(PARTITION BY creation_date::date) AS quest_count\
FROM stackoverflow.posts p\
LEFT JOIN stackoverflow.post_types t ON p.post_type_id=t.id\
WHERE \
type = 'Question' \
AND \
(creation_date::date BETWEEN '2008-11-01' AND '2008-11-18')\
) quest;

## 3. Сколько пользователей получили значки сразу в день регистрации? Выведите количество уникальных пользователей.

---
SELECT COUNT(DISTINCT u.id)\
FROM stackoverflow.users u\
JOIN stackoverflow.badges b ON u.id=b.user_id\
WHERE b.creation_date::date = u.creation_date::date;

## 4. Сколько уникальных постов пользователя с именем Joel Coehoorn получили хотя бы один голос?

---
SELECT COUNT(DISTINCT v.post_id)\
FROM stackoverflow.users u\
JOIN stackoverflow.posts p ON u.id=p.user_id\
JOIN stackoverflow.votes v ON p.id=v.post_id\
WHERE u.display_name = 'Joel Coehoorn';

## 5. Выгрузите все поля таблицы vote_types. 
Добавьте к таблице поле rank, в которое войдут номера записей в обратном порядке. \
Таблица должна быть отсортирована по полю id.

---
SELECT *,\
RANK() OVER(ORDER BY id DESC) AS rank\
FROM stackoverflow.vote_types\
ORDER BY id;

## 6. Отберите 10 пользователей, которые поставили больше всего голосов типа Close. 
Отобразите таблицу из двух полей: идентификатором пользователя и количеством голосов. \
Отсортируйте данные сначала по убыванию количества голосов, \
потом по убыванию значения идентификатора пользователя.

---
SELECT DISTINCT v.user_id,\
COUNT(v.vote_type_id) AS voice_count\
FROM stackoverflow.votes v\
JOIN stackoverflow.vote_types t ON t.id=v.vote_type_id\
WHERE t.name = 'Close'\
GROUP BY v.user_id\
ORDER BY voice_count DESC, v.user_id DESC\
LIMIT 10;

## 7. Отберите 10 пользователей по количеству значков, полученных в период с 15 ноября по 15 декабря 2008 года включительно.
Отобразите несколько полей:
- идентификатор пользователя;
- число значков;
- место в рейтинге — чем больше значков, тем выше рейтинг.\
Пользователям, которые набрали одинаковое количество значков, присвойте одно и то же место в рейтинге.\
Отсортируйте записи по количеству значков по убыванию, а затем по возрастанию значения идентификатора пользователя.

---
SELECT DISTINCT b.user_id,\
COUNT(b.id) AS znak,\
DENSE_RANK() OVER(ORDER BY COUNT(b.id) DESC) AS raiting\
FROM stackoverflow.badges b\
WHERE b.creation_date::date BETWEEN '2008-11-15' AND '2008-12-15'\
GROUP BY user_id\
ORDER BY COUNT(b.id) DESC, user_id\
LIMIT 10;

## 8. Сколько в среднем очков получает пост каждого пользователя?
Сформируйте таблицу из следующих полей:
- заголовок поста;
- идентификатор пользователя;
- число очков поста;
- среднее число очков пользователя за пост, округлённое до целого числа.
- Не учитывайте посты без заголовка, а также те, что набрали ноль очков.

---
WITH post AS\
(SELECT title,\
user_id,\
score\
FROM stackoverflow.posts\
WHERE title IS NOT NULL\
AND score != 0)

SELECT title,\
user_id,\
score,\
ROUND(AVG(score) OVER(PARTITION BY user_id)) AS avg_score\
FROM post;


## 9. Отобразите заголовки постов, которые были написаны пользователями, получившими более 1000 значков. 
Посты без заголовков не должны попасть в список.

---
WITH znak AS\
(SELECT DISTINCT user_id,\
 COUNT(*) OVER(PARTITION BY user_id)\
FROM stackoverflow.badges\
),

foucent AS\
(SELECT *\
FROM znak\
WHERE count >= 1000)

SELECT p.title\
FROM foucent f\
JOIN stackoverflow.posts p ON f.user_id=p.user_id\
WHERE p.title IS NOT NULL;

## 10. Напишите запрос, который выгрузит данные о пользователях из США (англ. United States). 
Разделите пользователей на три группы в зависимости от количества просмотров их профилей:
- пользователям с числом просмотров больше либо равным 350 присвойте группу 1;
- пользователям с числом просмотров меньше 350, но больше либо равно 100 — группу 2;
- пользователям с числом просмотров меньше 100 — группу 3.\
Отобразите в итоговой таблице идентификатор пользователя, количество просмотров профиля и группу. \
Пользователи с нулевым количеством просмотров не должны войти в итоговую таблицу.

---
WITH usa AS\
(SELECT DISTINCT id,\
views\
FROM stackoverflow.users\
WHERE location LIKE '%United States%'\
AND views != 0)

SELECT id,\
views,\
CASE\
----WHEN views >= 350 THEN 1\
----WHEN views < 350 AND views >= 100 THEN 2\
----WHEN views < 100 THEN 3\
END \
FROM usa

## 11. Дополните предыдущий запрос. Отобразите лидеров каждой группы — пользователей, 
которые набрали максимальное число просмотров в своей группе. \
Выведите поля с идентификатором пользователя, группой и количеством просмотров. \
Отсортируйте таблицу по убыванию просмотров, а затем по возрастанию значения идентификатора.

---
WITH usa AS\
(SELECT DISTINCT id,\
----views,\
CASE\
----WHEN views >= 350 THEN 1\
----WHEN views >= 100 AND  views < 350 THEN 2\
----WHEN views < 100 THEN 3\
END AS user_group\
FROM stackoverflow.users\
WHERE views != 0 AND LOCATION LIKE ('%United States%'))

SELECT id,\
views,\
user_group\
FROM\
(SELECT *,\
----DENSE_RANK() OVER (PARTITION BY user_group ORDER BY views DESC)\
FROM usa\
ORDER BY user_group, views DESC) AS rating\
WHERE dense_rank = 1

ORDER BY 2 DESC, 1 ASC

## 12. Посчитайте ежедневный прирост новых пользователей в ноябре 2008 года. 
Сформируйте таблицу с полями:
- номер дня;
- число пользователей, зарегистрированных в этот день;
- сумму пользователей с накоплением.

---
WITH c AS\
(SELECT EXTRACT(DAY FROM creation_date) as day_number,\
COUNT(id) us\
FROM stackoverflow.users\
WHERE creation_date::date BETWEEN '2008-11-01' AND '2008-11-30'\
GROUP BY day_number\
ORDER BY day_number)

SELECT *,\
SUM(us) OVER( ORDER BY day_number)\
FROM c

## 13. Для каждого пользователя, который написал хотя бы один пост, 
найдите интервал между регистрацией и временем создания первого поста. \
Отобразите:
- идентификатор пользователя;
- разницу во времени между регистрацией и первым постом.

---
WITH dat AS\
(SELECT DISTINCT u.id,\
u.creation_date as reg,\
MIN(p.creation_date) as first_post\
FROM stackoverflow.users u\
JOIN stackoverflow.posts p ON p.user_id=u.id\
GROUP BY u.id, reg)

SELECT dat.id,\
first_post - reg AS diff\
FROM dat

## 14. Выведите общую сумму просмотров постов за каждый месяц 2008 года. 
Если данных за какой-либо месяц в базе нет, такой месяц можно пропустить.\ 
Результат отсортируйте по убыванию общего количества просмотров.

---
SELECT DATE_TRUNC('month', creation_date)::date as month, \
SUM(views_count) as sum_views\
FROM stackoverflow.posts\
WHERE creation_date::date BETWEEN '2008-01-01' AND '2008-12-31'\
GROUP BY month\
ORDER BY sum_views DESC

## 15. Выведите имена самых активных пользователей, которые в первый месяц после регистрации 
(включая день регистрации) дали больше 100 ответов. Вопросы, которые задавали пользователи, не учитывайте. \
Для каждого имени пользователя выведите количество уникальных значений user_id. \
Отсортируйте результат по полю с именами в лексикографическом порядке.

---
SELECT display_name as name,\
COUNT(DISTINCT u.id) as count_answer\
FROM stackoverflow.users u\
JOIN stackoverflow.posts p ON p.user_id=u.id\
JOIN stackoverflow.post_types pt ON pt.id=p.post_type_id\
WHERE pt.type = 'Answer'\
AND p.creation_date::date BETWEEN u.creation_date::date AND u.creation_date::date + INTERVAL '1 month'\
GROUP BY name\
HAVING COUNT(u.id) > 100\
ORDER BY name

## 16. Выведите количество постов за 2008 год по месяцам. 
Отберите посты от пользователей, которые зарегистрировались в сентябре 2008 года \
и сделали хотя бы один пост в декабре того же года. \
Отсортируйте таблицу по значению месяца по убыванию.

--- 
WITH srez AS\
(\
SELECT DISTINCT u.id\
JOIN stackoverflow.posts p ON p.user_id=u.id\
WHERE u.creation_date::date BETWEEN '2008-09-01' AND '2008-09-30'\
AND (p.creation_date::date BETWEEN '2008-12-01' AND '2008-12-31')\
)

SELECT DATE_TRUNC('month', pp.creation_date)::date as month,\
COUNT(srez.id)\
FROM srez\
JOIN stackoverflow.posts pp ON pp.user_id=srez.id\
WHERE pp.creation_date::date BETWEEN '2008-01-01' AND '2008-12-31'\
GROUP BY month\
ORDER BY month DESC

## 17. Используя данные о постах, выведите несколько полей:
- идентификатор пользователя, который написал пост;
- дата создания поста;
- количество просмотров у текущего поста;
- сумму просмотров постов автора с накоплением.\
Данные в таблице должны быть отсортированы по возрастанию идентификаторов пользователей, \
а данные об одном и том же пользователе — по возрастанию даты создания поста.

---
SELECT user_id,\
creation_date,\
views_count,\
SUM(views_count) OVER(PARTITION BY user_id ORDER BY creation_date)\
FROM stackoverflow.posts\
ORDER BY 1

## 18. Сколько в среднем дней в период с 1 по 7 декабря 2008 года включительно пользователи взаимодействовали с платформой? 
Для каждого пользователя отберите дни, в которые он или она опубликовали хотя бы один пост. \
Нужно получить одно целое число — не забудьте округлить результат.

---
WITH action AS\
(\
SELECT user_id,\
COUNT(DISTINCT creation_date::date) AS act_days\
FROM stackoverflow.posts\
WHERE creation_date::date BETWEEN '2008-12-01' AND '2008-12-07'\
GROUP BY 1\
)

SELECT ROUND(AVG(act_days))\
FROM action

## 19. На сколько процентов менялось количество постов ежемесячно с 1 сентября по 31 декабря 2008 года? 
Отобразите таблицу со следующими полями:
- номер месяца;
- количество постов за месяц;
- процент, который показывает, насколько изменилось количество постов в текущем месяце по сравнению с предыдущим.\
Если постов стало меньше, значение процента должно быть отрицательным, если больше — положительным. \
Округлите значение процента до двух знаков после запятой.

---
WITH post AS\
(\
SELECT EXTRACT(MONTH FROM creation_date) as month,\
COUNT(id) post_count\
FROM stackoverflow.posts\
WHERE creation_date::date BETWEEN '2008-09-01' AND '2008-12-31'\
GROUP BY month\
)

SELECT *,\
ROUND((post_count - LAG(post_count) OVER(ORDER BY month))::numeric * 100.0 / LAG(post_count) OVER(ORDER BY month), 2) AS share\
FROM post p

## 20. Выгрузите данные активности пользователя, который опубликовал больше всего постов за всё время. 
Выведите данные за октябрь 2008 года в таком виде:
- номер недели;
- дата и время последнего поста, опубликованного на этой неделе.

---
WITH act_user AS\
(SELECT DISTINCT user_id,\
COUNT(id) as post_count\
FROM stackoverflow.posts\
GROUP BY user_id\
ORDER BY post_count DESC\
LIMIT 1)

SELECT DISTINCT EXTRACT(WEEK FROM p.creation_date) as week_number,\
LAST_VALUE(p.creation_date) OVER(PARTITION BY EXTRACT(WEEK FROM p.creation_date::date) ORDER BY EXTRACT(WEEK FROM p.creation_date))\
FROM stackoverflow.posts p\
JOIN act_user u ON u.user_id=p.user_id\
WHERE p.creation_date::date BETWEEN '2008-10-01' AND '2008-10-31'
