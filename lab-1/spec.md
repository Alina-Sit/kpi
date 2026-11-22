# Spec: Онлайн-бібліотека з книгами, бронюванням і купівлею

Тип усіх id: number. Поля-зв'язки (author_id, library_id тощо) лишаються в атрибутах і в діаграмі мають називатися так само.

Типи полів (крім id):
- string: User.name, User.email, Author.name, Book.title, DigitalVersion.file_url, Collection.title, ReaderGroup.title, Library.name, Library.address, PickupPoint.type, PickupPoint.address, Delivery.type, Payment.kind, ReadingProgress.word_marker;
- number: Book.price, Copy.sold_price, Delivery.speed, Delivery.cost, Payment.amount, Fine.amount, ReadingProgress.page;
- boolean: ReaderGroup.is_public, Fine.is_paid, ReadingProgress.is_finished;
- date: Reservation.start_date, Reservation.end_date, Reservation.returned_at;
- datetime: User.created_at, Purchase.created_at, Delivery.received_at, Payment.paid_at.
Усі поля `*_id` мають тип number, як і id.

## Сутності й атрибути

- **User**: id, name, email, created_at. Особистий кабінет окремою сутністю не є, це відображення даних користувача.
- **Author**: id, name, user_id (необов'язковий).
- **Book**: id, title, price, author_id.
- **Copy**: id, book_id, library_id (необов'язковий), purchase_id (необов'язковий), sold_price (необов'язкове).
- **DigitalVersion**: id, book_id, file_url.
- **Collection**: id, title, author_id.
- **ReaderGroup**: id, title, owner_id, is_public.
- **Library**: id, name, address.
- **PickupPoint**: id, type (library / shop / post), address, library_id (необов'язковий).
- **Reservation**: id, user_id, copy_id, start_date, end_date, returned_at (необов'язкове, порожнє, поки книгу не повернули).
- **Purchase**: id, user_id, created_at.
- **Delivery**: id, type, speed (днів), cost, received_at (необов'язкове, порожнє, поки не отримано), pickup_point_id, reservation_id (необов'язковий), purchase_id (необов'язковий). Значення type: `slow` (30-60 днів), `express` (платна, 1-14 днів), `self_pickup` (самовивіз, вартість 0).
- **Payment**: id, delivery_id, amount, kind (prepayment / surcharge), paid_at.
- **Fine**: id, reservation_id, amount, is_paid.
- **ReadingProgress**: id, user_id, book_id, is_finished, page, word_marker.

## Зв'язки

- Користувач може стати автором: при завантаженні першої книги з'являється запис Author з його user_id. Автор має багато книг, у книги один автор.
- Книга має багато примірників або жодного (0..*) і максимум одну цифрову версію (0..1).
- Примірник числиться максимум в одній бібліотеці (0..1). Після купівлі він вказує на купівлю.
- Колекція–книга, група–книга і «збережені колекції» це «багато до багатьох» без власних даних, тож зв'язки прямі.
- Користувач має багато броней, бронь стосується одного примірника. Купівля містить багато фізичних примірників.
- Доставка належить рівно одній броні або купівлі й завжди їде в один пункт видачі. Платіж належить доставці.
- Штраф належить одній броні (максимум один на бронь).
- ReadingProgress зв'язує користувача й книгу (один запис на пару).

## Критерії

1. Максимум 3 активні броні на людину (активна, поки returned_at порожнє).
2. Несплачений штраф (is_paid = false) блокує нові броні.
3. У примірника заповнене рівно одне з двох: library_id або purchase_id. sold_price заповнене тоді й лише тоді, коли є purchase_id, і дорівнює Book.price на момент купівлі.
4. Бронювати й купувати можна лише примірник із library_id, що не має активної броні. Одна активна бронь на примірник.
5. Примірник створює бібліотека, і library_id при створенні її.
6. Термін броні (end_date мінус start_date) не більше 6 місяців.
7. Штраф створюється, якщо книгу не повернули до end_date або повернули після end_date. Одна бронь має максимум один штраф, його amount фіксований: бібліотека задає суму при створенні, далі вона не змінюється.
8. Для кожної броні й купівлі рівно одна доставка. У доставки заповнене рівно одне з reservation_id або purchase_id, а pickup_point_id обов'язковий.
9. Для пункту типу library library_id обов'язковий, для shop і post порожній.
10. Самовивіз: пункт лише типу library, cost = 0, speed = 0, і це бібліотека, де примірник числиться на момент оформлення. Для купівлі з кількох примірників лише якщо всі в одній бібліотеці.
11. Платіж потрібен, лише коли сума до сплати більше 0. Для купівлі вона складається з доставки й книг, для броні лише з доставки.
12. Коли доставку броні в пункт типу library отримано (received_at), library_id примірника стає цією бібліотекою.
13. Купівля складається щонайменше з одного примірника, і всі вони фізичні.
14. Будь-який користувач може додати власну книгу й цифрову версію, а Author.user_id унікальне (один запис автора на користувача). Для класики без акаунта автора книгу, цифрову версію й ціну додає бібліотека. Колекцію створює лише користувач із записом автора.
15. ReadingProgress: один запис на пару користувач + книга, лише для книг із цифровою версією.
16. Модель у 3NF, окрім sold_price (історичний факт). Немає сполучних таблиць для «багато до багатьох» без власних даних. Типи id однакові, назви полів у діаграмі й spec збігаються.