# SQL

The **Structured Query Language** ([SQL](https://wikipedia.org/wiki/SQL)) is a
standard declarative language used in relational database managers.

A relational database, as we've seen already, is a set of relations and
constraints. Usually, the mathematical definition is replaced by the more
convenient and practical notion of tables, rows and columns; sql helps us
interact with those concepts. We can create and delete tables, update or
retrieve rows and much more.

The sql model is not a perfect match of the relational model. One important
deviation is that in sql a table is not a set of tuples, given that it allows
repeated tuples, so it's more like a list of tuples.

There are a lot of RDBMS, open source and/or commercial. I'll use
[sqlite](https://sqlite.org) since it's easy to install and use. It has a
simple architecture: through a terminal program or a library, sql statements
are interpreted and executed and all the data is stored in one file.

## Tables and constraints

The fist thing needed to structure data following a relational paradigm are
tables. In them, columns are defined which implement relation attributes, each
one having a data type. To create a table in a database the `create table`
statement is used.

    
    
    create table books (
      book_id integer primary key autoincrement,
      title text not null
    );
    

This will create the table `books` with columns `book_id` and `title`. The
first column was defined as an `integer` primary key. The `autoincrement` part
means each new row will automatically get an unique integer. The second
column, named `title`, is a `text` field and was defined as `not null`.

When creating a table, it's possible to define several attributes as a primary
key, by using `primary_key(attr_1,...,attr_n)`.

To relate two tables, a column can be defined as a **`foreign key`**. For
example

    
    
    create table bookshop(
        address text not null,
        name text not null,
        bs_id integer primary key autoincrement,
        owner_id integer not null,
        foreign key (owner_id)
            references owner (owner_id)
            on delete cascade
            on update no action
    );
    

Here we created the `bookshop` table. This table has 4 columns: `address`,
`name`, `bs_id` and `owner_id`. The most interesting one is the latter; the
`foreign key (owner_id)` defines that column as a foreign key, `references
owner (owner_id)` says that the foreign key in question (`owner_id`) points to
the `owner_id` column in the `owner` table; the `on delete cascade` means that
when the row the foreign key points to is deleted, the row storing the foreign
key also gets deleted:

    
    
    +---------------------------+
    |BOOKSHOP                   |
    +-------+----+-----+--------+
    |address|name|bs_id|owner_id|
    +-------+----+-----+--------+
    | ----  | -- |  1  |    2   |--+
    +-------+----+-----+--------+  |   +--------------------+
                                   |   |OWNER               |
                                   |   +------+----+--------+
                                   |   |gender|name|owner_id|
                                   |   +------+----+--------+
                                   |   |  --  | -- |   1    |
                                   |   +------+----+--------+
                                   +-> |  M   | -- |   2    |
                                       +------+----+--------+
    

By defining a column as a foreign key, the database kowns that it has to
enforce the **referential integrity constraint**. This means that every
foreign key has to point to another existing row.

Another possible constraint in a table is the `unique` constraint, which makes
it impossible to have two rows with the same value in the **`unique`**
attribute. Just as with `primary key`, `unique` can be used in several
columns. When used in several columns, like

    
    
    create table book (
        book_id integer primary key autoincrement,
        title text not null unique,
            author_name text not null unique,
            unique(title, author_name)
    );
    

it means that two rows can't have the same values in the `title` and
`author_name` simultaneously. The `unique` constraint doesn't apply to the
`null` value.

The **`not null`** is a type of constraint in the values that an attribute can
have. More general constraints can be defined by using **`check`** ; it can be
used following a column definition or after the columns were defined:

    
    
    create table book (
        book_id integer primary key autoincrement,
        title text not null unique,
        price real not null check(price >= 0 and price < 10000)
    );
    

or

    
    
    create table book (
        book_id integer primary key autoincrement,
        title text not null unique,
        price real not null check( and price < 10000),
            discount real not null,
        check(price >= 0 and discount < price)
    );
    

The **`drop`** statement is used to remove a table from the database:

    
    
    drop table table_name;
    

Some other operations could be needed before dropping the database: in
sqlite3, the foreign key check has to be disabled and the foreign keys
pointing to the to-be-dropped table changed to `null`.

To change a table definition, **`alter table`** can be used. For example, to
change a table name:

    
    
    alter table table_name
    rename to new_name;
    

or to add another column to the table:

    
    
    alter table table_name
    add column column_name type ...;
    

## SQL Data types

The values in the rows, it's attributes, have a type. A type is a set of
possible values.

This are the datatypes supported by the `sqlite3` database:

  * `null` represents a missing value
  * `integer` are integer numbers (0, 1, 2, 3,..., -1, -2, -3, ...)
  * `real` are floating point numbers
  * `text` is a string of characters, without maximum length
  * `blob` is a string of bits, without maximum length

So, any attribute will be defined as one of those types. Other databases, like
Postgres or MySQL, define different datatypes.

In the sql standard, types can either be predefined, constructed or user
defined. The [predefined types](https://wikipedia.org/wiki/SQL) are:

  * Character Types

    * char (character)
    * varchar (variable size characters)
    * clob (character large object)
  * Binary types

    * binary
    * varbinary (variable binary)
    * binary large object (blob)
  * Numeric Types

    * numeric, decimal, smallint, integer, bigint (eact numeric tupes)
    * float, real, double precision (exact numeric types)
  * date, time, timestamp (Datetime Types)

  * intervar (interval type)

  * boolean

  * xml

## Changing and adding data

In sql adding data is made through the **`insert`** statement. Let's take the
`book` table in the previous section:

    
    
        create table book (
        book_id integer primary key autoincrement,
        title text not null 
    );
    

Let's insert a row into that table:

    
    
    insert into book (title, book_id) values ("Lord of the flies", 1);
    

And to add several rows in the same command:

    
    
    insert into (title, book_id) values
        ("The power and the glory", 2),
        ("Gestarescala", 3),
        ("Ubik", 4),
        ("Slaughterhouse 5", 5),
        ("1984", 6),
        ("Animal Farm", 7),
        ("In cold blood", 8),
        ("After dark", 9),
        ("Sobre heroes y tumbas", 10),
        ("El túnel", 11),
        ("La invención de morel", 12),
        ("Diario de la guerra del cerdo", 13);
    

The values to `insert` can be taken from another table by using `select`:

    
    
    insert into table_name
        [(column,..., column)]
    select statement;
    

For example, let's assume we have a table `book2` which contains book titles
and author names; then we decided to split those tables into two: a `book`
table and an `author` table, to accomodate the requirement that some books
could have more than one author. To accomplish that, after creating the
tables, we could fill the `book` table by using

    
    
    insert into book(title)
    select title from book2;
    

which will fill the table `book` with the titles from `book2`.

If instead of adding new rows, we need to change existing ones, the
**`update`** statement is used:

    
    
    update table_name
    set column=expression,..., column=expression
    [where condition];
    

So, for changing the title of a book, we could do:

    
    
    update book
    set title = "1984"
    where title = "1985";
    

For deleting one or several rows there's a statement called **`delete`** :

    
    
    delete from table_name
    [where condition];
    

Let's remove all books with an empty title, since they are probably mistakes:

    
    
    delete from book
    where title = '';
    

**Be careful** : if no `where` condition filters the selection all the rows
are deleted.

## Example database

In order to show how to retrieve data, this section introduces an example
database.

We're going to model a book selling tool. Bookshops can sign up to sell books
and clients can buy them and leave reviews (one per book). Each bookshop will
have a unique owner. Each book can have one or more authors. Let's create an
Entity Relational Model with this information:

    
    
    (gender, name, _owner_id_)                  (gender, name, _author_id_)
    |                          wrote         m  |
    +-----+                     /\--------------+------+
    |Owner| (title, _book_id_)  \/              |Author|
    +-----+                  |  |               +------+
      |n       (price)       |  |n
      |        |             +----+                   +------+
      /\owns   |  bought    m|Book|0   reviewed_by    |Review|
      \/       ---/\---------+----+--------/\---------+------+
      |           \/        0       1      \/      1         |
      |m      0   ||                       |             (text, _rev_id_)
    +--------+----++---------+0           0|1     
    |BookShop|p             n| +------+----+           +-------+
    +--------+               +-|Client|-----------/\---|Profile|--(_profile_id_, bio, website, phone)
    |                          +------+ 1         \/  1+-------+
    (address, name, _bs_id_)          |
                                      (gender, name, _client_id_, email)
    

And the Relational Model

    
    
    BOOK(title, _book_id_)
    
    AUTHOR(gender, name, _author_id_)
    
    REVIEW(time, text, _rev_id_)
    
    PROFILE(_profile_id_, bio, website, phone)
    
    CLIENT(gender, name, email, _client_id_, *profile)
    
    BOOKSHOP(address, name, _bs_id_, *owner_id)
    
    OWNER(gender, name, _owner_id_)
    
    WRITTEN_BY([*book_id, *author_id])
    
    SELLS([*book_id, *bs_id], stock)
    
    BOUGHT([*bs_id, *client_id, *book_id], price)
    
    REVIEWED_BY([*rev_id, *book_id, *client_id])
    

To implement this in a relational database, we just need to create a table for
each one of the schemas and add the appropriate constraints.

The following code implements that model in `sqlite3`:

    
    
    create table book(
        title text not null,
        book_id integer primary key autoincrement
    );
    
    insert into book (title, book_id) values
        ("Lord of the flies", 1),
        ("The power and the glory", 2),
        ("Gestarescala", 3),
        ("Ubik", 4),
        ("Slaughterhouse 5", 5),
        ("1984", 6),
        ("Animal Farm", 7),
        ("In cold blood", 8),
        ("After dark", 9),
        ("Sobre heroes y tumbas", 10),
        ("El túnel", 11),
        ("La invención de morel", 12),
        ("Diario de la guerra del cerdo", 13);
    
    
    create table author(
        gender text not null,
        name text not null,
        author_id integer primary key autoincrement
    );
    
    insert into author (gender, name, author_id) values
        ("M", "Graham Greene", 1),
        ("M", "Philip K. Dick", 2),
        ("M", "Kurt Vonnegut", 3),
        ("M", "George Orwell", 4),
        ("M", "William Golding", 5),
        ("M", "Truman Capote", 6),
        ("M", "Haruki Murakami", 7),
        ("M", "Ernesto Sábato", 8),
        ("M", "Adolfo Bioy Casares", 9);
    
    
    create table owner(
        gender text not null,
        name text not null,
        owner_id integer primary key autoincrement
    );
    
    insert into owner (gender, name, owner_id) values
        ("M", "Jorge Luis Borges", 1),
        ("F", "Beverly Atlee Clearly", 2),
        ("M", "Jorge of Burgos", 3);
    
    
    create table profile(
        profile_id integer primary key autoincrement,
        website text not null default "",
        phone text not null default "",
        bio text not null default ""
    );
    
    insert into profile (website, phone, bio) values
        ("", "", ""),
        ("", "", ""),
        ("", "", "");
    
    
    create table client(
        gender text not null,
        name text not null,
        email text not null unique,
        client_id integer primary key autoincrement,
        profile integer not null,
        foreign key (profile)
            references profile (profile_id)
            on delete cascade
            on update no action
    );
    
    
    insert into client (gender, name, client_id, email, profile) values
        ("M", "Umberto Eco", 1, "umberto@echo.com", 1),
        ("M", "Gary Stu", 2, "gary@stu.com", 2),
        ("F", "Mary Sue", 3, "mary@sue.com", 3);
    
    
    create table review(
        time text not null,
        text text not null,
        rev_id integer primary key autoincrement
    );
    
    insert into review (time, text, rev_id) values
        ("2020-05-021 21:05:00.000", "Very good!", 1),
        ("2020-05-021 21:06:00.000", "Not very good...", 2),
        ("2020-05-021 21:07:00.000", "Excelent.", 3),
        ("2020-05-021 21:08:00.000", "Recommended.", 4);
    
    
    create table bookshop(
        address text not null,
        name text not null,
        bs_id integer primary key autoincrement,
        owner_id integer not null,
        foreign key (owner_id)
            references owner (owner_id)
            on delete cascade
            on update no action
    );
    
    insert into bookshop (address, name, bs_id, owner_id) values
        ("Buenos Aires", "El Ateneo", 1, 1),
        ("New York City", "The Strand", 2, 2),
        ("Greece", "Atlantis Books", 3, 3),
        ("Porto", "Livraria Lello", 4, 3);
    
    
    create table written_by(
        book_id integer not null,
        author_id integer not null,
        primary key (book_id, author_id),
        foreign key (book_id)
            references book (book_id),
        foreign key (author_id)
            references author (author_id)
    );
    
    insert into written_by (book_id, author_id) values
        (1, 5),
        (2, 1),
        (3, 2),
        (4, 2),
        (5, 3),
        (6, 4),
        (7, 4),
        (8, 6),
        (9, 7),
        (10, 8),
        (11, 8),
        (12, 9),
        (13, 9);
    
    
    create table sells(
        book_id integer not null,
        bs_id integer not null,
        stock integer not null,
        foreign key (bs_id)
            references bookshop (bs_id)
            on delete cascade
            on update no action,
        foreign key (book_id)
            references book (book_id)
            on delete cascade
            on update no action,
        primary key (book_id, bs_id)
    );
    
    insert into sells (book_id, bs_id, stock) values
        (1, 1, 100),
        (2, 2, 52),
        (3, 2, 60),
        (4, 1, 45),
        (5, 4, 123),
        (6, 4, 164),
        (7, 3, 200);
    
    
    create table bought (
        bs_id integer not null,
        client_id integer not null,
        book_id integer not null,
        price integer not null,
        primary key(bs_id, client_id, book_id)
        foreign key (bs_id)
            references bookshop (bs_id)
            on delete cascade
            on update no action,
        foreign key (book_id)
            references book (book_id)
            on delete cascade
            on update no action,
        foreign key (client_id)
            references client (book_id)
            on delete cascade
            on update no action
    );
    
    insert into bought (bs_id, client_id, book_id, price) values
        (1, 1, 1, 50),
        (2, 1, 2, 34),
        (3, 2, 7, 25),
        (4, 3, 3, 63),
        (1, 1, 2, 12),
        (2, 2, 3, 76),
        (3, 2, 12, 34),
        (4, 3, 12, 44),
        (1, 3, 3, 98),
        (2, 1, 12, 21),
        (3, 2, 10, 45),
        (4, 3, 5, 67);
    
    
    create table reviewed_by(
        rev_id integer not null,
        book_id integer not null,
        client_id integer not null,
        primary key (rev_id, book_id, client_id),
        foreign key (rev_id)
            references review (rev_id)
            on delete cascade
            on update no action,
        foreign key (book_id)
            references book (book_id)
            on delete cascade
            on update no action,
        foreign key (client_id)
            references client (client_id)
            on delete cascade
            on update no action
    );
    
    insert into reviewed_by (rev_id, book_id, client_id) values
        (1, 1, 1),
        (2, 3, 2),
        (3, 4, 1),
        (4, 5, 4);
    

To run this code with the `sqlite3` client, save the code into a file, let's
say `db.sql` and run

    
    
    sqlite3 db.db < db.sql
    

it will create the `db.db` file containing the database. I'll be using this
database to illustrate sql concepts.

## Querying the database

Just as in relational algebra, getting information from a relational database
involves composing transformations to groups of tables.

The most fundamental query statement is **`select`** :

    
    
    select [all|distinct] column_list
    from table_name alias [table_name alias,..., table_name alias]
    [where condition]
    [group by column list]
    [having condition]
    [order by column_name [asc|desc]];
    

As explained before, the `select` statement basically does a `σ(π)` in
relational algebra. For example:

    
    
    select name, client_id
    from client c
    where c.gender = 'F';
    

means retrieving all the female clients' names and database id's saved in the
database. In relational algebra we would do

    
    
    π[name, client_id](σ[gender = 'F'](client))
    

which would return a table with two columns, `name` and `client_id`:

    
    
    Mary Sue|3
    

And if we filter by male clients, by doing

    
    
    select name, client_id
    from client c
    where c.gender = 'M';
    

we get

    
    
    Umberto Eco|1
    Gary Stu|2
    

The `where` part on sql statements filters rows according to some comparison,
it works like the `σ` in relational algebra. Its form is:

    
    
    expression comparison_operator expression
    

where `comparison_operator` can be `=`, `!=`, `<`, `>`, `<=` or `>=`. Also,
they can be joined by logical operators `and`, `or`, etc.:

    
    
    expression1 comparison_operator expression2 logical_op
    expression3 comparison_operator expression4 ...
    

By adding **`distinct`** to the clause, you avoid repeated tuples from
appearing:

    
    
    select distinct name, client_id
    from client c
    where c.gender = 'M';
    

If there's no `where` condition, then there's no filter, so all the rows are
used. If more than one table is specified in the `from`, then the cartessian
product is taken.

The order in which the results appear respect the order in the original table,
but by using **`order by`** you can sort the result:

    
    
    select name, client_id
    from client c
    where c.gender = 'M'
    order by name;
    

You can also set the order to ascending or descending, by using `asc` or
`desc` respectively.

The `null` value is a bit special in comparisons, since `null` could mean
unknown, every comparison will going to return `null`. This needs a three
value logic:

    
    
    +-----+------+-------+------+
    | AND | true | false | null |
    +-----+------+-------+------+
    |true | true | false | null |
    +-----+------+-------+------+
    |false| false| false | false|
    +-----+------+-------+------+
    |null | null | false | null |
    +-----+------+-------+------+
    
    +-----+------+-------+------+
    | OR  | true | false | null |
    +-----+------+-------+------+
    |true | true | true  | null |
    +-----+------+-------+------+
    |false| true | false | false|
    +-----+------+-------+------+
    |null | true | null  | null |
    +-----+------+-------+------+
    
    +-----+------+
    | NOT |      |
    +-----+------+
    |true | false|
    +-----+------+
    |false| true |
    +-----+------+
    |null | null |
    +-----+------+
    

## Joins

The relations between tables are very important for the relational model. It's
very usual that a query involves two or more tables related through foreign
keys. For example, if we want a list of the books an author wrote, we need to
involve three tables: `book`, `author` and `written_by`.

Lets get all the books written by Philip K. Dick. Since we didn't define the
`name` field as a `primary key` or `unique`, we can't use the name to look for
it, so, let's use `author_id` (in this case, the value is 2). We'll take the
`written by` table and the `book` table, then filter the cartesian product
between both by keeping only those rows with the same `author_id` field.

    
    
    select b.title
    from written_by w, book b
    where w.author = 2 AND w.book_id = b.book_id;
    

which gives back

    
    
    Gestarescala
    Ubik
    

Another common pattern is joining two tables to get a list of the relationship
tuples. For example, let's get all the tuples of the `owns` relationship:

    
    
    select o.name, b.name
    from owner o, bookshop b
    where o.owner_id = b.owner_id;
    

getting the answer

    
    
    Jorge Luis Borges|El Ateneo
    Beverly Atlee Clearly|The Strand
    Jorge of Burgos|Atlantis Books
    Jorge of Burgos|Livraria Lello
    

This pattern is so useful and common that it has its own name: an `inner
join`:

    
    
    select o.name, b.name
    from owner o inner join bookshop b on o.owner_id = b.owner_id;
    

which gives the same result:

    
    
    Jorge Luis Borges|El Ateneo
    Beverly Atlee Clearly|The Strand
    Jorge of Burgos|Atlantis Books
    Jorge of Burgos|Livraria Lello
    

The last `on o.owner_id = b.owner_id` can be replaced by `using(owner_id)` as
a simplification.

That operation is called an inner join. `sqlite3` supports two other kinds of
joins: **left join** and **cross join**. For example, if we add an owner
without a bookstore pointing to it, for example:

    
    
    insert into owner(gender, name) values
        ("F", "No One");
    

Now, applying a left join:

    
    
    select o.name, b.name
    from owner o left join bookshop b using(owner_id);
    

which results in

    
    
    Jorge Luis Borges|El Ateneo
    Beverly Atlee Clearly|The Strand
    Jorge of Burgos|Atlantis Books
    Jorge of Burgos|Livraria Lello
    No One|
    

Note that the owner No One has a `null` in the corresponding column in the
right side of the join, that is, `bookshop`.

The **cross join** is the equivalent of the cartesian product. For example:

    
    
    select *
    from client
    cross join booksreview;
    

gives the not very useful result:

    
    
    M|Umberto Eco|1|2020-05-021 21:05:00.000|Very good!|1
    M|Umberto Eco|1|2020-05-021 21:06:00.000|Not very good...|2
    M|Umberto Eco|1|2020-05-021 21:07:00.000|Excelent.|3
    M|Umberto Eco|1|2020-05-021 21:08:00.000|Recommended.|4
    M|Gary Stu|2|2020-05-021 21:05:00.000|Very good!|1
    M|Gary Stu|2|2020-05-021 21:06:00.000|Not very good...|2
    M|Gary Stu|2|2020-05-021 21:07:00.000|Excelent.|3
    M|Gary Stu|2|2020-05-021 21:08:00.000|Recommended.|4
    F|Mary Sue|3|2020-05-021 21:05:00.000|Very good!|1
    F|Mary Sue|3|2020-05-021 21:06:00.000|Not very good...|2
    F|Mary Sue|3|2020-05-021 21:07:00.000|Excelent.|3
    F|Mary Sue|3|2020-05-021 21:08:00.000|Recommended.|4
    

There are other kinds of joins, such as the full outer join, which puts a null
in the rows which don't match, be it right or left. `sqlite3` doesn't
implement this type of join.

## Set Operations

There are three set operations implemented in `sqlite3`: `union`, `except` and
`intersect`. The **`union`** operator combines the result from two querys into
a single table. For this to be possible, the two querys have to be compatible,
that is, they have the same amount of columns and each column have the same
types (or types which can be casted to the other).

There are two versions of the `union` operator: `union` and `union all`. The
first one gets rid of repeated rows, while the second doesn't.

Let's use `union` to get all the names of the authors, clients and owners in
our database:

    
    
    select name from author union select name from client union select name from owner;
    
    Adolfo Bioy Casares
    Beverly Atlee Clearly
    Ernesto Sábato
    Gary Stu
    George Orwell
    Graham Greene
    Haruki Murakami
    Jorge Luis Borges
    Jorge of Burgos
    Kurt Vonnegut
    Mary Sue
    Philip K. Dick
    Truman Capote
    Umberto Eco
    William Golding
    

The **`intersect`** operation takes the result of two querys and returns those
rows which occur in both querys. Again, the result of the querys have to be
compatible in the same way as before. Let's change the last query but using
`intersect`:

    
    
    select name from author union select name from client union select name from owner;
    

The result is empty since no row is repeated.

The **`except`** operator is similar to the set difference: given two querys
`A` and `B`, `A except B` returns the rows in `A` that are not in `B`. For
example:

    
    
    select name from author except select name from client;
    
    Adolfo Bioy Casares
    Ernesto Sábato
    George Orwell
    Graham Greene
    Haruki Murakami
    Kurt Vonnegut
    Philip K. Dick
    Truman Capote
    William Golding
    

## Views

A view is an abstract table constructed as a `select` query. When reading the
data in a view, the client is actually reading the result of a query on actual
tables (called base tables).

A view can be defined in `sqlite3` as:

    
    
    create view authors_and_books as
    select book_id, author_id, title, name
    from written_by
    inner join book using(book_id)
    inner join author using(author_id);
    

The `authors_and_books` table represents the result of the two joins which, of
course, is also a table. The view can be queried:

    
    
    select * from authors_and_books;
    

returns the same rows as

    
    
    select book_id, author_id, title, name
    from written_by
    inner join book using(book_id)
    inner join author using(author_id);
    
    1|5|Lord of the flies|William Golding
    2|1|The power and the glory|Graham Greene
    3|2|Gestarescala|Philip K. Dick
    4|2|Ubik|Philip K. Dick
    5|3|Slaughterhouse 5|Kurt Vonnegut
    6|4|1984|George Orwell
    7|4|Animal Farm|George Orwell
    8|6|In cold blood|Truman Capote
    9|7|After dark|Haruki Murakami
    10|8|Sobre heroes y tumbas|Ernesto Sábato
    11|8|El túnel|Ernesto Sábato
    12|9|La invención de morel|Adolfo Bioy Casares
    13|9|Diario de la guerra del cerdo|Adolfo Bioy Casares
    

The view concept gives a lot of flexibility. For example, we can query the
view in a simple line to repeat the search for all of Philip K. Dick books in
our database:

    
    
    select title from authors_and_books
    where author_id = 2;
    
    Gestarescala
    Ubik
    

## Functions and Triggers

The following model is taken from the database example

    
    
                             (_profile_id_, bio, website, phone)
                             |
    +------+                +-------+
    |Client|-----------/\---|Profile|
    +------+ 1         \/  1+-------+
       |
    (gender, name, _client_id_, email)
    

It states that for every `Client` there's a `Profile`. So, if a `Client` is
added to the database, a `Profile` needs to be added first. The software
client which is using the database, such as a web application, needs to be
aware of this and follow those steps.

Another approach is to tell the database manager to do it automatically, using
a **trigger**. A trigger is a programmed response to an event in a database
table that executes a series of statements. An event can be an `update`,
`insert` or `delete` operation. Triggers are associated to tables, that is,
they are activated when modifications happen on certain table. The trigger
statements can be executed `before`, `after` or `instead of` the event.

In the case above, the trigger has to be fired when a new `Client` is created,
which corresponds with an `insert into client` statement. To create the
trigger:

    
    
    create trigger create_profile_for_new_user after insert on client begin
    insert into profile(website, phone, bio) values ("created", "in the", "trigger!");
    update client set profile=last_insert_rowid() where client_id = new.client_id;
    end;
    

The `last_insert_rowid()` is a function provided by `sqlite3` which returns
the row id of the most recently inserted row in the current conection. Let's
test it by creating a new `Client`

    
    
    insert into client (gender, name, email, profile) values ('', 'new', 'new@user.com', null);
    

and querying to see if the corresponding profile was created

    
    
    select c.name, p.website, p.phone, p.bio from client c inner join profile p on c.profile = p.profile_id;
    Umberto Eco|||
    Gary Stu|||
    Mary Sue|||
    new|created|in the|trigger!
    

The `Cient` allows a `null` value in the `profile` column, which we don't
actually want, since it has total participation. This choice was taken because
`sqlite3` lacks the `instead of` trigger in a table, so we need to let the
database create a record for the user first, using `null` in profile and then
the trigger fixes it. Another solution could be to store a foreign key to
`Client` in `Profile` instead of the other way arround; but the `Profile`
table has the potential to be used by other tables, such as `Author` or
`Owner`, so the design is more extensible this way.

# Transactions

A [transaction](https://wikipedia.org/wiki/Database_transaction) is an
undivisable unit of work in a database. A set of operations can be made atomic
by wrapping them in a transaction so they will be performed as if they were
inseparable; if one fails the entire operation fails and the database is
unchanged.

There are four properties a database has to implement in its transactions
(collectively known as ACID):

  * Atomic: all operations within a transaction are executed as if they were one. If one fails, the database remains unchanged and if the transaction ends succesfully all operations were performed.

  * Consistent: changes produced by the operations inside a transaction doesn't leave the database in an inconsistent state.

  * Islation: transactions are isolated from each other and can be performed concurrently (one doesn't collide with the other).

  * Durable: the changes in the database produced by the operations within a successful transaction are stored in the database permanently.

In `sqlite3` most sql statements are implicitly wrapped inside a transaction.
The ones involving write operations can be done one at a time, but the ones
only reading data can be executed simultaneously.

To create a transaction in `sqlite3`, wrap the statements you want to be
atomic between `begin transaction` and `commit transaction`. A classical
example is transfering an amount from one account to another:

    
    
    create table account (
        account_id integer not null primary key,
        balance real not null default 0,
        check(balance >= 0)
    );
    
    begin transaction
        update account
        set balance = balance - 100
        where account_id = 1;
    
        update account
        set balance = balance + 100
        where account_id = 2;
    commit transaction
    

If the operations in this simple example wouldn't be inside a transaction,
another client trying to interact with the same rows could create a
concurrency problem.

