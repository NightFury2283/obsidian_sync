[[Питон Практикум]]
[[SQL]]
[[SQLite]]
## JOIN

Конструкция `FROM video_products, slogans` соединяет таблицы, а `WHERE` — фильтрует получившуюся выборку.

По стандарту SQL92 принято отделять фильтрацию от условий соединения таблиц с помощью инструкции `JOIN` (англ. «соединять»):

```sql
-- Верни все поля
SELECT *
-- из таблицы video_products
FROM video_products
-- ...но перед этим присоедини записи из таблицы slogans так, чтобы
-- в записях значения полей video_products.slogan_id и slogans.id были равны.
JOIN slogans ON video_products.slogan_id = slogans.id;
```


Результат тот же, а запрос выглядит лучше:

- соединение и фильтрация разделены; это упрощает понимание запроса;
- условия соединения таблиц содержатся в блоке `ON`, это уменьшает вероятность ошибки.

В классическом SQL существует пять типов `JOIN`; SQLite поддерживает только три из них. Но с помощью ловкости и смекалки неподдерживаемые типы можно заменить другими запросами.


### Внутреннее пресечение: INNER JOIN

На первый тип `JOIN` вы уже полюбовались: в примере выше был как раз `INNER JOIN` (или просто `JOIN`, без титулов).

Объединение таблиц через `INNER JOIN` можно представить схематически:


![[Pasted image 20250506221400.png]]


Запрос через `JOIN` можно сделать и к трём таблицам. Присоединим таблицу **product_types**:

```sql
SELECT video_products.title,
       slogans.slogan_text,
       product_types.title
FROM video_products
JOIN slogans ON video_products.slogan_id = slogans.id
JOIN product_types ON video_products.type_id = product_types.id;
```


> Оператор `INNER JOIN` включает в результирующую таблицу только те записи, в которых выполняется условие, заданное в `ON`.


### Левое внешнее соединение: LEFT OUTER JOIN

При обработке запроса `LEFT OUTER JOIN` объединяемые таблицы условно называют **«левая»** и **«правая»**. «Левая» — та, которая вызвана в блоке `FROM`, «правая» — та, что указана после ключевого слова `JOIN`. «Правых» таблиц может быть и несколько.

В этом запросе `OUTER` — не обязательное слово. Можно использовать сокращённую запись: `LEFT JOIN`.

При выполнении запросов с `LEFT JOIN` возвращаются **все строки левой таблицы**. Данными из **правой таблицы** дополняются только те строки левой таблицы, для которых выполняются условия соединения, описанные после оператора `ON`. Для недостающих данных вместо строк правой таблицы вставляется `NULL`.

```sql
SELECT video_products.title,
       slogans.slogan_text
FROM video_products
LEFT JOIN slogans ON video_products.slogan_id = slogans.id;
```

```bash
('Безумные мелодии Луни Тюнз', None)
('Весёлые мелодии', None)
('Хороший, плохой, злой', "For Three Men The Civil War Wasn't Hell. It Was Practice!")
('Последний киногерой', "This isn't the movies anymore")
('Она написала убийство', 'Tonight on Murder She Wrote')
```

«Безумные мелодии Луни Тюнз» и «Весёлые мелодии» остались в выборке, поскольку `LEFT JOIN` **возвращает все записи левой таблицы без исключения**. Но у этих фильмов нет слогана, и вместо слогана вернулся `NULL` (`None` в переводе на язык Python).

Графически связь `LEFT JOIN` изображается так:

![[Pasted image 20250506222743.png]]


### RIGHT OUTER JOIN

`RIGHT JOIN` — это такое же объединение, как и `LEFT JOIN`, но **выводятся все записи из правой таблицы**, а к ним добавляются только те данные из левой таблицы, в которых есть ключ объединения.

Напишем запрос через `RIGHT JOIN`:


```sql
SELECT video_products.title,
       product_types.title
FROM video_products
RIGHT JOIN product_types ON video_products.type_id = product_types.id;
```

```bash
('Безумные мелодии Луни Тюнз', 'Мультсериал')
('Весёлые мелодии', 'Мультсериал')
('Последний киногерой', 'Фильм')
('Хороший, плохой, злой', 'Фильм')
('Она написала убийство', 'Сериал')
(None, 'Мультфильм')
```

Будут выведены все записи из правой таблицы (из **product_types**) и связанные с ними записи из левой (из **video_products**). В таблице **video_products** нет ни одной записи, ссылающейся на тип «Мультфильм», поэтому вместо значения `video_products.title` в этой строке стоит `None`.

Графическое отображение `RIGHT JOIN`:

![[Pasted image 20250506223040.png]]


SQLite поддерживает `RIGHT JOIN` [с 25 июня 2022 года](https://www.sqlite.org/releaselog/3_39_0.html) (версия 3.39.0). В более ранних версиях при попытке выполнить `RIGHT JOIN` возникнет ошибка.


##### Выход есть: можно применить «разрешённый» в SQLite `LEFT JOIN`, поменяв таблицы местами. В нашем случае нужно назвать **product_types** главной (левой), а **video_products** присоединить через `LEFT JOIN`.


### FULL OUTER JOIN

При запросе **FULL (OUTER) JOIN** выводятся **все записи из объединяемых таблиц**. Те записи, у которых запрошенные значения совпадают, — выводятся парами, у остальных недостающее значение заменяется на `NULL` (Python выведет `None`).

Другими словами, **FULL JOIN == LEFT JOIN + RIGHT JOIN.**

В классическом SQL сработает такой запрос:

```sql
SELECT video_products.title,
       slogans.slogan_text
FROM video_products
FULL JOIN slogans ON video_products.slogan_id = slogans.id;
```


В sqlite есть FULL JOIN, но если используем старую версию:

```sqlite
При возникновении такой ошибки можно пойти другим путём: выполнить два запроса `LEFT JOIN` и выполнить команду `UNION` — она объединяет данные из нескольких результирующих таблиц в одну:
```

```sqlite
SELECT video_products.title,
       slogans.slogan_text
FROM video_products
LEFT JOIN slogans ON video_products.slogan_id = slogans.id
UNION
SELECT video_products.title,
       slogans.slogan_text
FROM slogans
LEFT JOIN video_products ON video_products.slogan_id = slogans.id;
```


![[Pasted image 20250508221636.png]]


### CROSS JOIN

Объединение таблиц через `CROSS JOIN` возвращает декартово произведение таблиц — каждая запись левой таблицы объединится с каждой записью правой. Параметр `ON` при запросах `CROSS JOIN` не применяется.

```sql
SELECT video_products.title,
       slogans.slogan_text
FROM video_products
CROSS JOIN slogans;
```

```
На практике `CROSS JOIN` применяется не очень часто, но и он может быть полезен. Например, если в одной таблице сохранён список жидкостей, а во второй — список возможной тары для расфасовки, от маленькой баночки до цистерны. В выборке получим все возможные комбинации «жидкость — тара».
```