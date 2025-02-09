# Базы данных

## Лабораторная работа № 2. Оператор SELECT языка SQL

### Теоретическая справка

Основой работы с базами данных является умение правильно составить запрос на фильтрацию данных. Для данной задачи используется оператор `SELECT`. Ниже представлен сокращенный синтаксис данного оператора. Более подробно можно прочитать в [документации](https://postgrespro.ru/docs/postgresql/15/sql-select).    

```sql
SELECT [ ALL | DISTINCT [ ON ( выражение [, ...] ) ] ]
    [ * | выражение [ [ AS ] имя_результата ] [, ...] ]
    [ FROM элемент_FROM [, ...] ]
    [ WHERE условие ]
    [ GROUP BY элемент_группирования [, ...] ]
    [ HAVING условие ]
    [ { UNION | INTERSECT | EXCEPT } [ ALL | DISTINCT ] выборка ]
    [ ORDER BY выражение [ ASC | DESC | USING оператор ] ]
    [ LIMIT { число | ALL } ]
    [ OFFSET начало [ ROW | ROWS ] ]
```

Оператор `SELECT` получает строки из базы данных и возвращает их в виде таблицы результатов запроса. Результат выполнения запроса может оказаться пустым множеством.   
Рассмотрим выполнение SQL запроса на фильтрацию данных по шагам.   

#### Фильтрация строк  

Первыми и обязательными элементами запроса являются ключевые слова `SELECT` и `FROM`. После ключевого слова `SELECT` указывается список столбцов, значения которых необходимо вывести. Возвращаемые столбцы могут быть получены непосредственно из базы данных или вычислены во время выполнения запроса. Для краткого обозначения того, что необходимо вывести все столбцы таблицы используют символ `*`.   

После ключевого слова `FROM` указывается имя таблицы, из которой будет производиться выборка данных. Данная таблица может быть реальной таблицей из базы данных, или виртуальной, полученной в результате подзапроса.   

```sql
SELECT * FROM student;
```

![example1](./img/select_all.png)

Данный запрос возвращает список всех студентов университета. Для того, чтобы вывести только ФИО студентов, необходимо вместо знака `*` указать названия требуемых полей.   

```sql
SELECT surname, name, patronymic FROM student;
```

![example2](./img/select_with_args.png)  

Следующий запрос   
```sql 
SELECT name FROM student;
```
Выведет все имена студентов, с учетом повторений. Для исключения повторов необходимо использовать ключевое слово `DISTINCT`.   

```sql
SELECT DISTINCT name FROM student;
```

#### Условия отбора строк

Для наложения условий поиска используется оператор `WHERE`. В качестве операторов сравнения используются операторы: `=`, `<>`, `>`, `>=`, `<`, `<=`.   

Например, чтобы найти всех студентов с именем «Сергей», возможно выполнить следующий запрос:   

```sql
SELECT surname, name, patronymic FROM student WHERE name = 'Сергей';
```

![select_sergey](img/select_sergey.png)

Обратите внимание, что сравниваемая строка заключается в одинарные кавычки. Об их использовании подробнее будет рассказано ниже.    

Аналогично можно найти всех студентов, у которых номер студенческого билета больше числа 850000.

```sql
SELECT surname, name, patronymic FROM student WHERE student_id > 850000;
``` 

![select_sergey](img/select_850000.png)


Для наложения нескольких условий возможно воспользоваться логическими операторами – `AND`, `OR`, `NOT`.  

Например, следующий запрос вернет ФИО студентов, чей день рождения попадает в диапазон с 25 июня 2002 до 25 июня 2003 года включительно.   

```sql
SELECT surname, name, patronymic FROM student WHERE birthday > '25/06/2002' AND birthday < '25/06/2003';
```

Аналогичный запрос можно сделать, используя ключевое слово `BETWEEN`  

```sql
SELECT surname, name, patronymic FROM student WHERE birthday BETWEEN '25/06/2002' AND '25/06/2003';
```

#### Проверка на содержание во множестве

С помощью ключевого слова `IN` возможно отобрать только те кортежи, заданный атрибут которых находится в указанном списке.   

Например, следующий запрос выводит список студентов, кто родился 23 ноября 2002 года или 09 февраля 2003 года.   

```sql
SELECT surname, name, patronymic FROM student WHERE birthday IN ('23/11/2001', '20/09/2001');
```


#### Поиск с использованием шаблона

Для наложения более сложных условий поиска возможно воспользоваться оператором поиска шаблонов `LIKE` и регулярными выражениями.   

```sql
строка LIKE шаблон [ESCAPE спецсимвол]
строка NOT LIKE шаблон [ESCAPE спецсимвол]
```

Оператор `LIKE` сравнивает анализируемую строку с заданным шаблоном и в случае совпадения отбирает эту строку. Для построения шаблона используются следующие спецсимволы:   

* `%` - любая последовательность символов
* `_` - строго один символ

Регулярные выражения POSIX   

* `^` - начало строки
* `$` - окончание строки
* `*` - повторение предыдущего символа любое число раз
* `\` - проверка наличия указанного после \ символа
* `|` - выбор одного из двух вариантов
* `~` - поиск с учетом регистра
* `[...]` - указание класса символов

При проверке по шаблону `LIKE` всегда рассматривается вся строка. Поэтому, если нужно найти последовательность символов где-то в середине строки, шаблон должен начинаться и заканчиваться знаками процента.   

Например, данный запрос позволяет найти всех студентов, чье имя начинается на букву ‘А’.   

```sql
SELECT surname, name, patronymic FROM student WHERE name LIKE 'А%'; 
```

Данный запрос вернет всех студентов, чье имя состоит из 5 символов и заканчивается на букву "я".   

```sql
SELECT surname, name, patronymic FROM student WHERE name LIKE '____я';
```

Рассмотрим пример с регулярными выражениями   

```sql
SELECT surname, name, patronymic FROM student WHERE surname ~ '^[ЛМН]';
```

Данный пример вернет всех студентов, чьи фамилии начинаются на буквы Л, М, Н. Символом ^ обозначается, что следующий символ будет первым в строке. В квадратных скобках указывается список допустимых символов. Аналогично можно записать через диапазон – [Л-Н]. Символ `~` указывает, что сравнение идет с учетом регистров.   

Дополним выражение таким образом, чтобы отбираемые строки оканчивались на буквосочетание `ин`.

```sql
SELECT surname, name, patronymic FROM student WHERE surname ~ '^[ЛМН].*ин$';
```

Символом `$` указывается, что предыдущие символы будут стоять в конце строки. Т.к. между ними и начальными символами могут находиться еще любое число символов, то обозначим их с помощью ‘.*’. В данном случае точка обозначает «любой символ», а звездочка продублирует его от 0 до любого числа раз.   

Если убрать символ окончания строки `$`, то будет производиться поиск строк, начинающихся на буквы Л, М, Н и содержащие в себе «ин».   

```sql
SELECT surname, name, patronymic FROM student WHERE surname ~ '^[ЛМН].*ин';
```

#### Вычисляемые столбцы  

На языке SQL возможно добавлять к итоговой выборке отдельные столбцы, значения которых будут вычисляться в процессе фильтрации. Для этого, к отбираемым столбцам после ключевого слова `SELECT` добавляется выражение, которое будет вычисляться для каждой строки.   

Например, рассчитаем возраст всех студентов вуза и выведем его вместе с ФИО. Для этого добавим еще один столбец и укажем в нем функцию age, позволяющую  вычислить разницу между двумя датами. В качестве точки отсчета выберем текущую дату, значение которой можно получить с помощью функции `CURRENT_DATE`.   

```sql
SELECT surname, name, patronymic, age(CURRENT_DATE, birthday) FROM student;
```

Для того, чтобы переименовать столбец age, воспользуемся ключевым словом `AS`. После него укажем новое название столбца – «Возраст».   

```sql
SELECT surname, name, patronymic, age(CURRENT_DATE, birthday) AS "Возраст" FROM student;
```

#### Агрегирование и группировка

Для проведения статистических вычислений в SQL существуют агрегатные функции. Данные функции принимают на вход множество значений и возвращают одно.  

Основные агрегатные функции:
* `AVG`: находит среднее значение
* `COUNT(*)`: находит количество строк в запросе
* `SUM`: находит сумму значений
* `MIN`: находит наxsименьшее значение
* `MAX`: находит наибольшее значение

Например, в данном примере рассчитывается средний оклад всех преподавателей вуза. Т.к. атрибут «Оклад» имеет тип money, то для применения агрегатной функции необходимо привести его к числовому значению. Для этого используется конструкция `::numeric`.   

```sql
SELECT AVG(salary::numeric)::numeric(10,2) AS "Average salary" FROM professor;
```

```sql
SELECT SUM(wage_rate) AS "Общее число ставок" FROM employment;
```

Однако, если потребуется рассчитать общее число ставок для каждого из направлений, то такой подход не сработает. Перед тем, как применять агрегирующую функцию необходимо сгруппировать кортежи по определенному признаку – в данном примере, по номеру направления. Затем необходимо применить функцию `SUM` к каждой из получившихся групп.    

Для группировки строк в SQL служит оператор GROUP BY. Данный оператор распределяет строки, образованные в результате запроса по группам, в которых значения некоторого столбца, по которому происходит группировка, являются одинаковыми. Группировку можно производить как по одному столбцу, так и по нескольким.   

Дополним запрос из предыдущего примера. В данном случае сумма будет рассчитана для каждого структурного подразделения. В подразделении 1 работают три преподавателя – поэтому их ставки просуммировались и результатом стало значение 1.00.    

```sql
SELECT structural_unit_number AS "Structural unit number", SUM(wage_rate) AS "Wage rate sum" FROM Employment GROUP BY structural_unit_number;
```

Рассмотрим еще один пример. В нем происходит подсчет числа сотрудников в каждом структурном подразделении.    

```sql
SELECT structural_unit_number AS "Structural unit number", count(*) AS "Number of employees" FROM employment GROUP BY structural_unit_number;
```

Для фильтрации строк перед группировкой использовалось ключевое слово `WHERE`. В случае, если нужно отфильтровать строки после неё – используется ключевое слово `HAVING`. 

Добавим в приведенный выше пример условие, чтобы число сотрудников выводилось только для подразделений, в которых более 2х преподавателей.  

```sql
SELECT structural_unit_number AS "Structural unit number", count(*) AS "Number of employees" FROM employment GROUP BY structural_unit_number HAVING count(*) > 2;
```

#### Сортировка

Для сортировки результата запроса необходимо использовать ключевое слово ORDER BY. После него указывается атрибуты, по которым производится сортировка. Далее указывается порядок с помощью слов `DESC` (в порядке убывания) и `ASC` (в порядке возрастания). По умолчанию строки сортируются по возрастанию, поэтому `ASC` можно опускать.   

```sql
SELECT structural_unit_number AS "Номер структурного подразделения", coun (*) AS "Число сотрудников" FROM employment GROUP BY structural_unit_number ORDER BY structural_unit_number;
```

```sql
SELECT structural_unit_number AS "Номер структурного подразделения", coun (*) AS "Число сотрудников" FROM employment GROUP BY structural_unit_number ORDER BY structural_unit_number DESC;
```

#### Оконные функции

При составлении запросов с использованием агрегирующих функций применялась группировка. Все строки с одинаковыми значениями, указанными после `GROUP BY` объединялись в одну и над каждой из данных групп совершалось определенное действие. Таким образом, число строк в результирующей таблице уменьшалось. Однако в некоторых случаях бывает полезным провести вычисления над группой и добавить вычисленное значение в качестве дополнительного столбца для каждой строки таблицы. Например, необходимо вывести студентов всех групп и пронумеровать в алфавитном порядке внутри каждой группы.  Для этого можно воспользоваться оконными функциями.   

В общем виде оконные функции выглядят следующим образом:   

```sql
SELECT
Название функции (столбец для вычислений) 
OVER (
        PARTITION BY столбец для группировки
        ORDER BY столбец для сортировки
)
```

В качестве функции может выступать одна из агрегатных функций (`SUM, COUNT, AVG, MIN, MAX`) или специальные функции, предназначенные для оконных вычислений.    

Приведем некоторые из них:   

* **ROW_NUMBER** – данная функция возвращает номер строки и используется для нумерации;
* **FIRST_VALUE** или **LAST_VALUE** — с помощью функции можно получить первое и последнее значение в окне. В качестве параметра принимает столбец, значение которого необходимо вернуть;
* **CUME_DIST** — вычисляет интегральное распределение (относительное положение) значений в окне;

После ключевых слов `PARTITION BY` необходимо указать поле, по которому будет производиться объединение в группы. Далее возможно отсортировать значения внутри каждой из групп.    

В итоге запрос для вычисления порядкового номера студента будет выглядеть следующим образом:    

```sql
SELECT row_number() OVER (partition by students_group_number ORDER BY student.surname), surname, name, patronymic FROM student;
```

Рассмотрим еще один пример, где каждому студенту добавляется поле, содержащее средний балл всей группы.    

```sql
SELECT DISTINCT	
	surname, 
	name, 
	patronymic, 
	avg(field_comprehension.mark) OVER(partition by students_group_number)::numeric(8,2), 
	student.students_group_number FROM student INNER JOIN field_comprehension ON field_comprehension.student_id = student.student_id;
```

#### Несколько важных замечаний

##### Экранирование кавычек

При использовании текстовых строк в запросах их необходимо обрамлять одинарными кавычками. Как, например, в следующем запросе:   

```sql
SELECT surname, name FROM student WHERE name = 'Анна';
```

Данный запрос выполнен не будет, т.к. СУБД распознает как строку только символ 'O'. Чтобы одинарная кавычка воспринималась как часть строки, а не указание на её окончание, в PostgreSQL предусмотрена возможность дублирования данной кавычки.   

```sql
SELECT surname, name FROM student WHERE surname = 'O''Brien' 
```

Способ, указанный выше, сработает.   

##### Типы данных в PostgreSQL

PostgreSQL предоставляет пользователям большой выбор встроенных типов данных. Ниже приведены основные из них. Более подробно можно прочитать в Главе 8 документации PostgreSQL.   


* **bigint** - знаковое целое из 8 байт
* **boolean** - логическое значение (true/false)
* **date** - календарная дата (год, месяц, день)
* **integer** - знаковое четырёхбайтное целое
* **json** - текстовые данные JSON
* **money** - денежная сумма
* **numeric [ (p, s) ]** - вещественное число заданной точности
* **real** - число одинарной точности с плавающей точкой (4 байта)
* **smallint** - знаковое двухбайтное целое
* **serial** - четырёхбайтное целое с автоувеличением
* **text** - символьная строка переменной длины
* **time [ (p) ] [ without time zone ]** - время суток (без часового пояса)
* **uuid** - универсальный уникальный идентификатор
* **varchar**(n) - строка ограниченной переменной длины

К основным целым числовым типам можно отнести типы **smallint(2 байта)**, **integer (4 байта)**, **bigint (8 байт)**. В большинстве случаем рекомендуется использовать тип **integer**.   

Для хранения дробных чисел произвольной точности возможно использовать тип **numeric**.    

`NUMERIC(точность, масштаб)`    

где точность – число значащих цифр (по обе стороны запятой), масштаб – число цифр после запятой. По умолчанию, тип numeric может хранить числа, содержащие до 1000 значащих цифр.    

Для хранения чисел с плавающей точкой используются типы **real** и **double**. Обратим внимание, что при арифметике чисел с плавающей точкой возможны ошибки округления и неточности при хранении данных, поэтому при необходимости точности вычислений рекомендуется использовать тип **numeric**.   

Данная проблема может стать критичной при хранении денежных сумм. Поэтому в случае хранения денежных единиц рекомендуется использовать тип **numeric** или **money**.   

Для хранения строковых типов ограниченного размера рекомендуется использовать тип **varchar(N)**, где N – максимальный размер строки. При необходимости хранения длинного текста возможно применять тип ***text***.    

Отдельно остановимся на типе, хранящим значения даты и времени. В PostgreSQL существует множество подобных типов. Наиболее популярными являются типы **date** и **time**. Все даты в PostgreSQL считаются по Григорианскому календарю, даже для времени до его введения.    

Данные о дате могут храниться разными способами. В соответствии со стандартом ISO 8601, рекомендуется хранить дату в формате «год-месяц-день». Например, 1998-08-26. Также возможно использовать следующую запись: месяц/день/год в режиме MDY или день/месяц/год в режиме DMY. Для проверки, в каком режиме работает PostgreSQL возможно выполнить команду:    

```sql
SELECT current_setting('datestyle');
```

Для переключения между режимами возможно использовать команду:   

```sql
SET datestyle TO "ISO, DMY";
```

В качестве первичного ключа очень часто используют последовательность из чисел от 1 и выше. Для хранения подобной последовательности используется тип **serial**. При создании переменной данного типа создается автоинкрементирующийся счетчик, увеличивающийся с добавлением новой записи. При вызове команды `INSERT` поле с данным типом возможно отметить значением `DEFAULT` или просто оставить пустым.   

В некоторых случаях необходимо, чтобы первичным ключом случил уникальный идентификатор, не повторяющийся ни в одной другой базе данных. Это может быть полезно, при объединении двух таблиц из различных баз данных. Таким идентификатором может служить **GUID** - глобальный уникальный идентификатор. Для его хранения в PostgreSQL используется тип **uuid**. **UUID** записывается в виде последовательности шестнадцатеричных цифр, разделённых знаками минуса на несколько групп: группа из 8 цифр, три группы из 4 цифр, группа из 12 цифр, что в сумме составляет 32 цифры и представляет 128 бит.    

> Пример **uuid**: 47a8229b-9e8a-0473-fd20-21c889da75bf

##### Преобразование типов

Преобразование типов в PostgreSQL возможно выполнить несколькими способами. Самый простой из них – напрямую указать тип данных через символ `::`.    

```sql
SELECT 1234::int;
```

```sql
SELECT 1234::text;
```

Аналогичного результата можно добиться, используя оператор `CAST`.   

```sql
SELECT CAST(1234 AS int);
```

```sql
SELECT CAST(1234 AS text);
```

### Задание

### Вопросы
