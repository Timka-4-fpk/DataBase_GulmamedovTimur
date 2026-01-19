# DataBase_GulmamedovTimur

## Лабораторные работы по дисциплине "базы данных"

### Тема: библиотечная система

**СУБД:** PostgreSQL

**Инструмент:** pgAdmin 4 ([https://sql.iscnet.ru/pgadmin4](https://sql.iscnet.ru/pgadmin4))

---

## Лабораторная работа №1

## Проектирование структуры базы данных

### Цель лабораторной работы

Спроектировать структуру базы данных (БД) на основе постановки задачи и анализа предметной области, при необходимости с учётом консультаций с преподавателем.

### Постановка задачи

В качестве предметной области выбрана **библиотечная информационная система**, предназначенная для хранения и обработки информации о книгах, авторах, читателях и фактах выдачи книг.

Задача выбрана из категории **задач, сгенерированных нейронной сетью**, и по сложности соответствует требованиям лабораторной работы.

### Требования к базе данных

* количество таблиц: от 3 до 5;
* база данных должна быть приведена как минимум к **третьей нормальной форме (3НФ)**;
* избыточность данных должна быть исключена.

### Структура базы данных

В результате проектирования были выделены следующие сущности:

1. **Authors** — авторы книг
2. **Books** — книги
3. **Readers** — читатели
4. **Rent** — выдача книг
5. **AuthorsBook** — связь авторов и книг (многие ко многим)

Таким образом, база данных содержит 5 таблиц, что соответствует требованиям задания.

### Нормализация

База данных приведена к **третьей нормальной форме**:

* каждая таблица описывает одну сущность;
* все неключевые атрибуты функционально зависят только от первичного ключа;
* связь «многие ко многим» реализована через отдельную таблицу.

### ER-диаграмма

На основе анализа предметной области была спроектирована ER-диаграмма, отражающая сущности, их атрибуты и связи между ними. Диаграмма использовалась в качестве логической модели при дальнейшей реализации БД.

<img src="Снимок экрана 2026-01-19 231157.png" alt="Скриншот проекта" width="600">

### Согласование

ER-модель была согласована с преподавателем до перехода к следующим лабораторным работам.

---

## Лабораторная работа №2

## Инсталляция БД на сервере. Логическая и физическая модели

### Цель лабораторной работы

Реализовать логическую и физическую модели базы данных и создать БД в PostgreSQL с использованием DDL-запросов.

### Логическая модель

Логическая модель была получена путём преобразования ER-диаграммы в набор взаимосвязанных таблиц с определёнными атрибутами и ключами.

### Физическая модель

Физическая модель реализована в СУБД PostgreSQL с использованием соответствующих типов данных, первичных и внешних ключей.

### DDL-запросы создания таблиц

```sql
CREATE TABLE authors (
    author_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    surname VARCHAR(100),
    date_of_birth DATE
);
```

```sql
CREATE TABLE books (
    book_id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    publish_year INT
);
```

```sql
CREATE TABLE readers (
    reader_id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    surname VARCHAR(100)
);
```

```sql
CREATE TABLE rent (
    rent_id SERIAL PRIMARY KEY,
    reader_id INT REFERENCES readers(reader_id),
    book_id INT REFERENCES books(book_id),
    rent_date DATE,
    return_date DATE
);
```

```sql
CREATE TABLE authorsbook (
    author_id INT REFERENCES authors(author_id),
    book_id INT REFERENCES books(book_id),
    PRIMARY KEY (author_id, book_id)
);
```

### Заполнение таблиц данными

```sql
INSERT INTO readers (name, surname) VALUES
('Иван', 'Иванов'),
('Пётр', 'Петров'),
('Анна', 'Смирнова'),
('Мария', 'Кузнецова');
```

```sql
INSERT INTO books (title, publish_year) VALUES
('Война и мир', 1869),
('Преступление и наказание', 1866),
('Мастер и Маргарита', 1967),
('Отцы и дети', 1862);
```

### SELECT-запрос с JOIN

```sql
SELECT r.name, r.surname, b.title
FROM rent rt
JOIN readers r ON rt.reader_id = r.reader_id
JOIN books b ON rt.book_id = b.book_id;
```

---

## Лабораторная работа №3

## Представления (VIEW) и хранимые процедуры

### Цель лабораторной работы

Реализовать представление для формирования выходных документов и встроенную процедуру с параметрами.

### Представление

```sql
CREATE VIEW rent_info AS
SELECT r.name, r.surname, b.title, rt.rent_date
FROM rent rt
JOIN readers r ON rt.reader_id = r.reader_id
JOIN books b ON rt.book_id = b.book_id;
```

### Хранимая функция

```sql
CREATE OR REPLACE FUNCTION get_rent_by_reader(p_reader_id INT)
RETURNS TABLE(book_title VARCHAR, rent_date DATE) AS $$
BEGIN
    RETURN QUERY
    SELECT b.title, rt.rent_date
    FROM rent rt
    JOIN books b ON rt.book_id = b.book_id
    WHERE rt.reader_id = p_reader_id;
END;
$$ LANGUAGE plpgsql;
```

---

## Лабораторная работа №4

## Анализ производительности запросов

### Цель лабораторной работы

Проанализировать производительность выполнения запросов и способы её улучшения.

### Генерация данных

Для каждой таблицы было сгенерировано по 20 000 записей с использованием SQL-циклов и функции `generate_series()`.

### Анализ выполнения запросов

Для анализа использовалась команда `EXPLAIN ANALYZE`, позволяющая изучить порядок выполнения операций и способы соединения таблиц.

### Оптимизация

Для ускорения выполнения запросов были добавлены индексы по внешним ключам.

---

## Лабораторная работа №5

## Триггеры

### Цель лабораторной работы

Реализовать триггеры для обеспечения целостности данных и ведения журнала изменений.

### Триггер каскадного удаления

```sql
CREATE OR REPLACE FUNCTION delete_rent_by_reader()
RETURNS TRIGGER AS $$
BEGIN
    DELETE FROM rent WHERE reader_id = OLD.reader_id;
    RETURN OLD;
END;
$$ LANGUAGE plpgsql;
```

```sql
CREATE TRIGGER trg_delete_reader
AFTER DELETE ON readers
FOR EACH ROW
EXECUTE FUNCTION delete_rent_by_reader();
```

### Триггер журналирования изменений

```sql
CREATE TABLE log_table (
    id SERIAL PRIMARY KEY,
    table_name TEXT,
    operation TEXT,
    operation_time TIMESTAMP
);
```

```sql
CREATE OR REPLACE FUNCTION log_changes()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO log_table(table_name, operation, operation_time)
    VALUES (TG_TABLE_NAME, TG_OP, NOW());
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

```sql
CREATE TRIGGER trg_log_rent
AFTER INSERT OR UPDATE OR DELETE ON rent
FOR EACH ROW
EXECUTE FUNCTION log_changes();
```

---

## Вывод

В ходе выполнения лабораторных работ была разработана, реализована и оптимизирована база данных в СУБД PostgreSQL. Выполнены все требования задания: проектирование структуры БД, реализация логической и физической моделей, создание представлений, процедур, анализ производительности и разработка триггеров.
