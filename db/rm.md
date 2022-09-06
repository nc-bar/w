### The relational model (RM)

The RM structures a database as a set of mathematical **relations**. Each
relation is described using a **relation schema** :

    
    
    `R(A1,...,An)`
    

`R` is the name of the relation and `Ai` is an attribute of `R` for every `i`.

Each attribute has a set of possible values, called it's **domain**. The
domain of an attribute `Ai` is expressed as `dom(Ai)`. The context is often
enough to show a attribute's domain:

    
    
                       atributes
                          ^
                          |
    PRODUCT(Name, Code, Price, Stock)
       ^
       |
     relation name
    

The amount of attributes in a relation is called it's **arity**. In the
example above, the arity is 4, because it's made up of 4 attributes.

A relation is a set of n-tuples `t=<v_1,...,v_n>`, where `v_i` is in
`dom(Ai)`. If `R` is a relation schema, `r(R)` represents the set of n-tuples
in a particular database. so if `R=PRODUCT`, a possible `r(PRODUCT)` is

    
    
    r(R) = {
        <"Thinkpad x201", 1, 10000, 6>,
        <"Pencil", 2, 1, 100>
    }
    

by definition `r(R)` is a subset of `dom(A1) X dom(A2) X...X dom(An)`, where X
is the cartesian product.

relations can be presented as tables. for example, a relation with schema
`BOOK(name, ISBN, author, id)` can be represented as:

    
    
    +-----------------------------------------------------------------+
    |BOOK                                                             |
    +------------------------+-------------------+---------------+----+
    |         name           |      ISBN         |    author     | id |
    +------------------------|-------------------|---------------+----+
    | Fahrenheit 451         | 84-7634-095-8     | Ray Bradbury  |  1 |
    +------------------------|-------------------|---------------+----+
    | The planet of the apes | 84-7634-078-8     | Pierre Boulle |  2 |
    +------------------------|-------------------|---------------+----+
    | 1984                   | 978-0-141-03614-4 | George Orwell |  3 |
    +------------------------+-------------------+---------------+----+
    

each attribute is placed in a column and each row is a 4-tuple arranged so
that the domains match. more generally, a schema of the form

    
    
    `R(A1,...,An)`
    

is represented as a table where its `i-th` column matches the `Ai` attribute;
the n-tuples in `r(R)` are the rows in the table.

the `BOOK` table

    
    
    +-----------------------------------------------------------------+
    |BOOK                                                             |
    +------------------------+-------------------+---------------+----+
    |         name           |      ISBN         |    author     | id |
    +------------------------|-------------------|---------------+----+
    | Fahrenheit 451         | 84-7634-095-8     | Ray Bradbury  |  1 |
    +------------------------|-------------------|---------------+----+
    | The planet of the apes | 84-7634-078-8     | Pierre Boulle |  2 |
    +------------------------|-------------------|---------------+----+
    | 1984                   | 978-0-141-03614-4 | George Orwell |  3 |
    +------------------------+-------------------+---------------+----+
    | 1984                   | 978-0-141-03614-4 | George Orwell |  3 |
    +------------------------+-------------------+---------------+----+
    

represents the relation:

    
    
    r(BOOK) = {
      <Fahrenheit 451, 84-7634-095-8, Ray Bradbury, 1>,
      <The planet of the apes, 84-7634-078-8, Pierre Boulle, 2>,
      <1984, 978-0-141-03614-4, George Orwell, 3>
    }
    

a relation represents a predicate about a set of tuples with the same arity
and domain. the predicate `p` of the schema `R(ai,...an)` can be defined as:

    
    
    p: Ai X .... X An --> {true, false}
        if <ai,...,an> in r(R) then p(<ai,...,an>) = true
        otherwise p(<ai,...,an>) = false
    

for a given `r(R)`.

example: `BOOK(Fahrenheit 451, 84-7634-095-8, Ray Bradbury, 1)` is true if and
only if `<Fahrenheit 451, 84-7634-095-8, Ray Bradbury, 1>` is in `r(BOOK)`.

a database, under the RM definition, is defined as a collection of schemas
such as:

    
    
    BOOK(name, ISBN, author, book_id)
    CLIENT(name, client_id)
    RENT(client_id, book_id, code)
    

a possible instance of the above database, presented in table form:

    
    
    +----------------------------------------------------------------------+
    |BOOK                                                                  |
    +----------------------------------------------------------------------+
    |         name           |      ISBN         |    author     | book_id |
    +------------------------|-------------------|---------------+---------+
    | Fahrenheit 451         | 84-7634-095-8     | Ray Bradbury  |    1    |
    +------------------------|-------------------|---------------+---------+
    | The planet of the apes | 84-7634-078-8     | Pierre Boulle |    2    |
    +------------------------|-------------------|---------------+---------+
    | 1984                   | 978-0-141-03614-4 | George Orwell |    3    |
    +------------------------------------------------------------+---------+
    
    +---------------------------------------------+
    |                   CLIENT                    |
    +---------------------------------------------+
    |         name           |      client_id     | 
    +------------------------|--------------------+
    | Mary Sue               |          1         |
    +------------------------|--------------------+
    | Gary Stu               |          2         |
    +------------------------+--------------------+
    
    +-----------------------------------------------+
    |                     RENT                      |
    +-----------------------------------------------|
    |   client_id    | book_id |        code        | 
    +----------------|---------|--------------------|
    |      1         |    1    |       123094       |
    +----------------|---------|--------------------|
    |      2         |    3    |       988356       |
    +-----------------------------------------------+
    

in terms of ER, a relation or table represents not only entities, but also
relationships. In fact, there is a correspondence between ER concepts and RM
concepts.

### constraints

not every value can be in a tuple and not any tuple can be in a relation. We
already described a table which is not permitted by the model:

    
    
    +-----------------------------------------------------------------+
    |BOOK                                                             |
    +------------------------+-------------------+---------------+----+
    |         name           |      ISBN         |    author     | id |
    +------------------------|-------------------|---------------+----+
    | Fahrenheit 451         | 84-7634-095-8     | Ray Bradbury  |  1 |
    +------------------------|-------------------|---------------+----+
    | The planet of the apes | 84-7634-078-8     | Pierre Boulle |  2 |
    +------------------------|-------------------|---------------+----+
    | 1984                   | 978-0-141-03614-4 | George Orwell |  3 |
    +------------------------+-------------------+---------------+----+
    | 1984                   | 978-0-141-03614-4 | George Orwell |  3 |
    +------------------------+-------------------+---------------+----+
    

Also, the values are, by the definition of the model to a set called the
`dom(A)`. It's also posible to set the values using a special value, called
`NULL`; attributes can be define as `NOT NULL`, meanning they don't allow the
`NULL` value.

It's also possible to define a subset of attributes on a relation R such that
no two tuples in `r(R)` can the same values in them at the same time. This is
called a **superkey**. Given that no two tuples can be repeated in `r(R)` we
can deduce that the super key `{A1,...,An}` is always a superkey in the
relation schema `R(A1,...,An)`. Of course it would be more useful to have a
smaller set of attributes as a superkey because it would give us the
possibility to find a specific tuple by knowing only somo of its attributes.
This type of constraint is called a uniqueness constraint.

In general a minimal set `S` related to a property `P` is such that `P(S)` is
true and if we take away any element from `S`, let's say `S' = S-{x}`, then
`P(S')` is not longer true. A **key** is a minimal superkey; this means that a
set of attributes of a relation `R` is a key if taking any attribute from `K`
would make it not a superkey. For example, in the relation schema

    
    
    EMPLOYEE(national_document, code, age, gender, salary)
    

the sets `{national_document, age}` and `{code, salary}` are superkeys, but
not keys, since they're not minimal. The sets `{national_document}` and
`{code}` are keys, because they have a uniqueness constraint and are minimal.
Of course not all keys are made up of only one attribute.

Out of all the keys a schema has, we can pick one to be the **primary key** ,
that is, the key we'll use to uniquely identify a tuple or row. To denote an
attribute as a primary key, I surround them by _, but the correct syntaxis
would be to underline the attribute name. For example

    
    
    EMPLOYEE(national_document, _code_, age, gender, salary)
    

would set the `EMPLOYEE` `code` as the key of this relation or table. In table
form, the same applies:

    
    
    +-----------------------------------------------------------------+
    |BOOK                                                             |
    +------------------------+-------------------+---------------+----+
    |         name           |      ISBN         |    author     |_id_|
    +------------------------|-------------------|---------------+----+
    

which sets `id` as the key. The key attributes can't be set as `NULL`,
otherwise, we couldn't refer to them. When a table contains a primary key of
another table, it's called a **foreign key** and it's used to refer to a row
on the other table.

The most important constraint in the relational model is the **referential
integrity** constraint. It states that a row referencing another via a foreign
key has to point to a valid row. So this database would violate the
referential integrity constraint:

    
    
    +----------------------------------------------------------------------+
    |BOOK                                                                  |
    +----------------------------------------------------------------------+
    |         name           |      ISBN         |    author     | book_id |
    +------------------------|-------------------|---------------+---------+
    | Fahrenheit 451         | 84-7634-095-8     | Ray Bradbury  |    1    |
    +------------------------|-------------------|---------------+---------+
    | The planet of the apes | 84-7634-078-8     | Pierre Boulle |    2    |
    +------------------------|-------------------|---------------+---------+
    | 1984                   | 978-0-141-03614-4 | George Orwell |    3    |
    +------------------------------------------------------------+---------+
    
    +---------------------------------------------+
    |                   CLIENT                    |
    +---------------------------------------------+
    |         name           |      client_id     | 
    +------------------------|--------------------+
    | Mary Sue               |          1         |
    +------------------------|--------------------+
    | Gary Stu               |          2         |
    +------------------------+--------------------+
    
    +-----------------------------------------------+
    |                     RENT                      |
    +-----------------------------------------------|
    |   client_id    | book_id |        code        | 
    +----------------|---------|--------------------|
    |      1         |    1    |       123094       |
    +----------------|---------|--------------------|
    |      2         |    4    |       988356       |
    +-----------------------------------------------+
    

because the tuple `<2,4, 988356>` references the `BOOK` with `book_id` 4 and
said tuple doesn't exist.

Together with the relation schema `R`, all the constraints further restrict
which `r(R)` are possible, to fit the model to the real object being model.

## ER to RM

The ER model describes data about entities and their relationships; the RM
model is used to structure related data. It's possible to translate an ER to a
practical RM database.

To begin, every entity will have a relation or table with the same attributes.
For example, the entity

    
    
      (author, ISBN, name, _book_id_)
      |
    +----+
    |BOOK|
    +----+
    

will be implemented as

    
    
    BOOK(name, ISBN, author, _book_id_)
    

Note the `_book_id_` attribute was chosen as a primary key.

The other important concepts in ER that we need to translate are the
relationships. Here, the translation changes according to the cardinality,
participation, and degree. To begin, let's examine binary 1:1 relationships.

    
    
      (name, _teacher_id_)
      |
    +-------+
    |TEACHER|
    +-------+
       0|1
        |
        /\head_of
        \/
        |
        |1
    +----------+
    |DEPARTMENT|------(name, _id_)
    +----------+
    

As stated before, each entity will have it's own table; also, we need to store
the relationship itself, that is, it's necessary to know which `TEACHER` is
the `head_of` which `DEPARTMENT`. A good way to do it is using a foreign key
in a table to reference the other participant in the relationship. In this
case, since not all `TEACHER`s are heads of a `DEPARTMENT`, it's a better idea
to add the foreign key into the `DEPARTMENT` table; otherwise a lot of
`TEACHER` records would need to use a `NULL` value. So, the schemes would be,
according to this rules:

    
    
    TEACHER(name, _teacher_id_)
    DEPARTMENT(name, _id_, *teacher_id)
    
    Note: *foreign key
    
    
    +---------------------------------------------+
    |                   TEACHER                   |
    +---------------------------------------------+
    |         name           |     teacher_id     | 
    +------------------------|--------------------+
    | Richard Feynman        |          1         |
    +------------------------|--------------------+
    | Donald Knuth           |          2         |
    +------------------------+--------------------+
    | Alan Turing            |          3         |
    +------------------------+--------------------+
    
    +-----------------------------------------------+
    |                    DEPARTMENT                 |
    +-----------------------------------------------|
    |      name      |    id   |     teacher_id     | 
    +----------------|---------|--------------------|
    |computer sicence|    1    |          1         |
    +----------------|---------|--------------------|
    |    physics     |    2    |          2         |
    +-----------------------------------------------+
    

If the binary relationship is of `1:N` cardinality, such as

    
    
      (name, _teacher_id_)
      |
    +-------+
    |TEACHER|
    +-------+
        |N
        |
        /\teaches_in
        \/
        |
        |1
    +----------+
    |DEPARTMENT|------(name, _d_id_)
    +----------+
    

Then, the solution is the same. This time, the foreign key has to go in the
entity in the `N` part of the relationship, for obvious reasons. The resulting
schemes would be:

    
    
    TEACHER(name, _teacher_id_, *d_id)
    DEPARTMENT(name, _d_id_)
    
    +---------------------------------------------+
    |                   DEPARTMENT                |
    +---------------------------------------------+
    |         name           |         d_id       | 
    +------------------------|--------------------+
    | Computer Science       |          1         |
    +------------------------|--------------------+
    | Physics                |          2         |
    +------------------------+--------------------+
    | Literature             |          3         |
    +------------------------+--------------------+
    
    +-----------------------------------------------+
    |                    TEACHER                    |
    +-----------------------------------------------|
    |      name      |   d_id  |     teacher_id     | 
    +----------------|---------|--------------------|
    |Richard Feynman |    1    |          1         |
    +----------------|---------|--------------------|
    |Donald Knuth    |    2    |          2         |
    +-----------------------------------------------+
    

Now, for `N:N` relationships pose the problem that no table can store a
foreign key, given that both entities can appear in several relationship
tuples. The way to solve it is creating a table for the relationship itself.
For example, if we now assume a `TEACHER` can teach in several `DEPARTMENT`s,
we have the ER model

    
    
      (name, _teacher_id_)
      |
    +-------+
    |TEACHER|
    +-------+
        |N
        |
        /\teaches_in
        \/
        |
        |N
    +----------+
    |DEPARTMENT|------(name, _d_id_)
    +----------+
    

that can be translated into the schemes

    
    
    DEPARTMENT(name, _d_id_)
    TEACHER(name, _teacher_id_)
    TEACHES_IN([*d_id, *teacher_id])
    

Note that in the table for the relationship, the attribute pair of both
primary keys is a primary key for the table.

    
    
    +---------------------------------------------+
    |                   DEPARTMENT                |
    +---------------------------------------------+
    |         name           |         d_id       | 
    +------------------------|--------------------+
    | Computer Science       |          1         |
    +------------------------|--------------------+
    | Physics                |          2         |
    +------------------------+--------------------+
    | Literature             |          3         |
    +------------------------+--------------------+
    
    +---------------------------+
    |          TEACHER          |
    +---------------------------+
    |      name      |teacher_id| 
    +----------------|----------+
    |Richard Feynman |    1     |
    +----------------|----------+
    |Donald Knuth    |    2     |
    +----------------|----------+
    |John Von Neuman |    3     |
    +----------------+----------+
    
    +---------------------------+
    |        TEACHES_IN         |
    +---------------------------+
    |      d_id      |teacher_id| 
    +----------------|----------+
    |        2       |    1     |
    +----------------|----------+
    |        1       |    2     |
    +----------------|----------+
    |        3       |    2     |
    +----------------+----------+
    |        3       |    1     |
    +----------------+----------+
    

In the case of unary or recursive relationships, the translation is not
complicated. Let's say we have

    
    
      (name, salary, hire_date, _number_)
      |
    +--------+
    |EMPLOYEE|--------+
    +--------+        |
        |             |
        |             |
        /\supervises  |
        \/            |
        |             |
        +-------------+
    

It can be implemented as

    
    
    EMPLOYEE(name, salary, hire_date, _number_, *supervisor)
    

in which the `*supervisor` is a foreign key to the same table.

The case of ternary relationships is very similar to the `N:M` binary
relationships. Each entity gets a table and another table for the relationship
is created, holding foreign keys to each of the entitties participating (the
three foreign keys together form the primary key, just as in the binary case).

## Relational Algebra

Having all your data structured and available is half the problem. The other
problem is getting it out in a useful way. For this, query languages are used
wich tell the database to perform a series of operations to retrieve data.

A theoreticaly important and influential language is the **Relational
Algebra**. In this language a set of operations are composed that take a
relation and produce another until, hopefuly, we get what we want. In another
words, the Relational Algebra is a set of functions that take a relation and
return another one.

Let's start with the **`σ`**. This operator takes a relationship and a list of
conditions, and returns the tuples in the relations that satisfy the
conditions. The form of the operator is

    
    
    σ[conditions](R)
    

where `R` is a relation and conditions wich a list of conditions that can be:

    
    
    attribute {=,≠,<,>,≤,≥} value in dom(attribute)
    attribute {=,≠,<,>,≤,≥} attribute with dom(attribute)=dom(attribute)
    

The list of conditions are joined by the logical operators `AND`, `OR`, `NOT`.

Let's use the following schemes for examples:

    
    
    BOOK(title, ISBN, _book_id_, price)
    AUTHOR(name, _author_id_)
    WRITTEN_BY([*book_id, *author_id])
    

And also the relations

    
    
    +--------------------------------------------------------------+
    |BOOK                                                          |
    +------------------------------------------------------+-------+
    |         title          |      ISBN         | book_id | price |
    +------------------------|-------------------|---------+-------+
    | Fahrenheit 451         | 84-7634-095-8     |    1    | 100   |
    +------------------------|-------------------|---------+-------+
    | The planet of the apes | 84-7634-078-8     |    2    | 200   |
    +------------------------|-------------------|---------+-------+
    | 1984                   | 978-0-141-03614-4 |    3    | 150   |
    +------------------------------------------------------+-------+
    
    +----------------------------------+
    |AUTHOR                            |
    +----------------------------------+
    |         name           |author_id|
    +------------------------|---------+
    | Ray Bradbury           |    1    |
    +------------------------|---------+
    | Pierre Boulle          |    2    |
    +------------------------|---------+
    | George Orwell          |    3    |
    +----------------------------------+
    
    +----------------------------------+
    |WRITTEN_BY                        |
    +----------------------------------+
    |    book_id             |author_id|
    +------------------------|---------+
    |        1               |    1    |
    +------------------------|---------+
    |        2               |    2    |
    +------------------------|---------+
    |        3               |    3    |
    +----------------------------------+
    

If we need a list of books that are worth at most 150 units, we can get it by

    
    
    σ[price ≤ 150](BOOK) = {<Fahrenheit 451,  84-7634-095-8, 1, 100>, <1984, 978-0-141-03614-4, 3, 150>}
    

The operator returns another relationship with the same attributes. If given a
relationship, we want to filter some attributes, we can use the projection
operator _`π`_ ; it takes a relation and a list of attributes and returns
another relation with the same tuples but containing only the attributes in
the list:

    
    
    π[title](BOOK) = {<Fahrenheit 451>, <The planet of the apes>, <1984>}
    

Of course, it's posible to compose these operators since they take and return
relations; for example, if we need all the titles which price is less or equal
that 150 we can do.

    
    
    π[title](σ[price ≤ 150](BOOK)) = {<Fahrenheit 451>, <1984>}
    

The useful **`ρ`** is the rename operator that can change the name of the
table and/or the attributes. For example, given the relation `R(A1,...,An)`:

    
    
    ρ[S(B1,...,Bn)](R)
    

changes the relation scheme to `S(B1,...,Bn)`. Ommiting either the name of the
relation or the attribute list changes only that.

Since relations are just sets by definition, the usual operators on sets are
available, with some restrictions. The cartesian product **X** , given the
relations `R(A1,...,An)` and `Q(B1,...,Bm)`, the new relation

    
    
    R(A1,...,An) X Q(B1,...,Bm)
    

has the scheme `S(A1,...,An,B1,...,Bm)`, and is composed of all the possible
concatenations of the tuples in each set; by concatenation I mean the
operation:

    
    
    <r1,...,rn>, <q1,...,qm> -> <r1,..., rn, q1,...,qm>
    

The other usual set operations also apply, like intersection, set difference
and union. To use them, we need the relations to be **type compatible**. Two
relations `R(A1,...,An)` and `S(B1,...,Bm)` are type compatible if
`dom(Ai)=dom(Bi)` and `n=m`. Otherwise, the resulting set wouldn't be a
relation necesarily.

A very important operator is join ( **`|X|`** ); it's equivalent to composing
**`X`** and **`σ`**. Join takes two relations and a list of conditions and
does:

    
    
    σ[conditions](R X S) ≡ R |X|[join condition] S
    

In other words, it takes all the posible concatenations and keeps the ones
which satisfy the conditions. The conditions only accept a list `AND`
conjunctions, not `OR` or `NOT`. Similarly, the **natural join** , symbolized
as ***** , does the same but has the implicit condition of equality to the
attribute with the same name in each relation, so

    
    
    R |X|[attr=attr] S ≡ R * S
    

here, `attr` is an attribute in both `R` and `S`. Also, the natural join
removes the duplicated column.

The Relational Algebra language inspired the popular SQL language which is a
standard implemented by most (all?) relational databases such as Postgres,
sqlite, MySQL, etc.

