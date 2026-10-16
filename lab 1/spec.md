# Специфікація: модель даних онлайн-бібліотеки

## Сутності й атрибути

### Author
- author_id: PK, number
- full_name: string
- birth_year: number

### Publisher
- publisher_id: PK, number
- name: string
- city: string

### Genre
- genre_id: PK, number
- name: string

### Book
- book_id: PK, number
- title: string
- isbn: string
- publication_year: number
- publisher_id: FK до Publisher, number
- genre_id: FK до Genre, number

### BookCopy
Фізичний примірник, який можна замовити.
- copy_id: PK, number
- inventory_number: string
- book_id: FK до Book, number

### DigitalEdition
Цифрова версія книги для читання онлайн.
- digital_edition_id: PK, number
- book_id: FK до Book, number
- file_format: string
- file_url: string

### Reader
- reader_id: PK, number
- full_name: string
- email: string
- registration_date: date

### Loan
Замовлення фізичного примірника.
- loan_id: PK, number
- copy_id: FK до BookCopy, number
- reader_id: FK до Reader, number
- loan_date: date
- due_date: date
- return_date: date, необов'язкове поле
- status: string
- delivery_address: string

### ReadingSession
Читання цифрового видання онлайн.
- session_id: PK, number
- reader_id: FK до Reader, number
- digital_edition_id: FK до DigitalEdition, number
- started_at: datetime
- last_page: number
- finished_at: datetime, необов'язкове поле

## Зв'язки
1. Publisher 1 — 0..* Book, мітка publishes: видавництво випускає багато книг, книга має одне видавництво.
2. Genre 1 — 0..* Book, мітка classifies: жанр має багато книг, книга належить одному жанру.
3. Author 1..* — 1..* Book, мітка writes: книга має одного або кількох авторів, автор пише одну або кілька книг. Це чистий зв'язок багато-до-багатьох без сполучної таблиці.
4. Book 1 — 0..* BookCopy, мітка has_copy: книга може мати фізичні примірники, а може не мати жодного, якщо вона існує лише онлайн.
5. Book 1 — 0..* DigitalEdition, мітка has_edition: книга може мати цифрові версії в різних форматах.
6. BookCopy 1 — 0..* Loan, мітка lent_in: один примірник може замовлятися багато разів.
7. Reader 1 — 0..* Loan, мітка makes: читач може мати багато замовлень.
8. Reader 1 — 0..* ReadingSession, мітка starts: читач може мати багато сесій читання.
9. DigitalEdition 1 — 0..* ReadingSession, мітка read_in: одну цифрову версію можуть читати багато разів.

Loan і ReadingSession є асоціативними сутностями, бо зв'язки між читачем і книгою несуть власні атрибути: дати, статус, адресу доставки, сторінку, на якій зупинився читач.

Допустимі значення Loan.status: ordered, shipped, active, returned.

## Критерії прийняття
1. Усі сутності у 3NF: немає повторюваних груп, часткових і транзитивних залежностей.
2. Усі id мають тип number, без змішування string і number.
3. Кожна сутність має PK. Кожен зв'язок «один-до-багатьох» має FK на стороні «багато» з вказаною кардинальністю. Зв'язок «багато-до-багатьох» (Author — Book) FK не має.
4. Немає сполучної таблиці для зв'язку Author і Book.
5. Назви полів у діаграмі збігаються зі spec: snake_case, без синонімів.
6. Діаграма не містить SQL-типів і фізичної схеми БД.
7. Фізичні та цифрові видання розділені на різні сутності, дані книги не дублюються в жодній із них.
8. Кожен зв'язок у діаграмі має мітку, яка збігається з міткою у spec.
9. Book.isbn і Reader.email унікальні.
10. Loan.status приймає лише значення ordered, shipped, active, returned.