# ER Model

The **Entity Relationship model**
([ER](https://wikipedia.org/wiki/Entity%E2%80%93relationship_model)) is a
method for abstracting parts of the real world into more manageable models. It
can also provide a more compact view of your database schema in the form of an
**Entity Relationship Diagram (ERD)**.

The fundamental concepts of the ER are the entities, their relationships and
attributes. The **entities** model a concept or object in the real world, such
as an organization or a person. Different entities have different properties
that are of interest to our problem, i.e. a person can have a name and last
name while a car has a color and a model. These properties are modeled by
**attributes**. Real world problems rarely involve only one entity or several
disconnected entities, they usually interconnect in several ways, and
**relationships** are how we model those connections.

In a database for a book catalog we could identify the book and client
entities, connected by a relationship between them stating which book was
rented by what client. The book entity could have the following attributes:
name, ISBN, author and id.

## Entities and Attributes

Since the ER is an abstract model for understanding information about reality
the choice of entities depends on the problem at hand. If we need to model a
library to know where all the books are then it makes sense to use the client
entity, so it's posible to store who borrowed some books; on the other hand,
if we also need to know the different sections a library has, then maybe the
section entity needs to be added.

When speaking of an entity we mean two things: a template of attributes
(called **entity type** ) and the datapoints that conform to that template
(collectively called **entity set** ). The term entity referes to both, which
can be confusing. When defining an entity in the ER model we usually define an
entity type by providing it's attributes and a name. To give an example let's
use the ERD:

    
    
      (author, ISBN, name, id)
      |
    +----+
    |BOOK|
    +----+
    

The box denotes the definition of an entity type and the names in parenthesis
its attribute list. This definition implies that the database contains data of
the form `<author, ISBN, name, id>` which form part of the entity set `BOOK`.
Again, but this time modeling the clients of the library:

    
    
      (name, client_id)
      |
    +------+
    |CLIENT|
    +------+
    

This model defines an entity `CLIENT` with attributes `name` and `client_id`.
From now on I'll probably make no distinction between entity set and entity
type, the difference should be clear in context.

The ER model describes two types of entities, a **weak entity** and a **strong
entity**. I've never been in a situation where I needed to use a weak entity,
so I wont explain the difference here.

Some attributes can be defined as **keys** which are those with different
values in different entities. This keys allow us to reference those specific
entities within an entity set. In the ERD we mark an attribute as a key by
surrounding it with _. (Note that the real ERD has a different
[syntaxis](https://wikipedia.org/wiki/Entity%E2%80%93relationship_model), but
is hard to transcribe it in plaintext). So, the `BOOK` entity could have the
`id` attribute be a key. This would be helpful if we want to reference a
specific book entity: we use one of its keys.

    
    
      (author, ISBN, name, _id_)
      |
    +----+
    |BOOK|
    +----+
    

The following entity set would conform to the previous model:

    
    
    <Ray Bradbury, Fahrenheit 451, 84-7634-095-8, 1>
    <Pierre Boulle, The planet of the apes, 84-7634-078-8, 2>
    <George Orwell, 1984, 978-0-141-03614-4, 3>
    

but this one doesn't:

    
    
    <Ray Bradbury, Fahrenheit 451, 84-7634-095-8, 1>
    <Pierre Boulle, The planet of the apes, 84-7634-078-8, 2>
    <George Orwell, 1984, 978-0-141-03614-4, 2>
    

since the `id` of vale 2 is repeated, and `id` is a key.

An entity can have more than one key:

    
    
      (author, _ISBN_, name, _id_)
      |
    +----+
    |BOOK|
    +----+
    

The following entity set would conform to the previous model:

    
    
    <Ray Bradbury, Fahrenheit 451, 84-7634-095-8, 1>
    <Ray Bradbury, The machineries of joy, 84-7634-078-8, 2>
    <George Orwell, 1984, 978-0-141-03614-4, 3>
    

but this one doesn't:

    
    
    <Ray Bradbury, Fahrenheit, 451 84-7634-095-8, 1>
    <Ray Bradbury, Fahrenheit, 84-7634-078-8, 2>
    <George Orwell, 1984, 978-0-141-03614-4, 3>
    

because `<Ray Bradbury, Fahrenheit>`, is a key set of attributes and are
repeated in the entity set.

Several attributes can conform a key, such as `<author,name>`, I'll use [] to
mark those attributes:

    
    
      ([author, name], ISBN, _id_)
      |
    +----+
    |BOOK|
    +----+
    

If an author can have more than one book with the same title, the last model
wouldn't be appropiate.

The attributes of an entity are the information about them we want to store;
their defining characteristics according to our needs.

If we are modeling a database for a bussiness, then we probably need to store
entities such as employees and products, but. About an employee we might need
information such as their full name, salary, hiring date, employee number,
etc.; from a product, it's name, internal code, price, stock, etc. These
properties are represented as attributes in the respective entities:

    
    
      (name, salary, hire_date, _number_)
      |
    +--------+
    |EMPLOYEE|
    +--------+
    
      (name, _code_, price, stock)
      |
    +-------+
    |PRODUCT|
    +-------+
    

This model represents databases such as:

    
    
    <Gabriel García Marquez, 100, 10/10/2010, 100>
    <Antonio Machado, 100, 20/4/2012, 150>
    <Jorge Luis Borges, 100, 22/6/2014>
    
    <La mala Hora, 1, 20, 100>
    <Antología Poética, 2, 15, 150>
    <El Aleph, 3, 30, 10>
    

Some attributes could have unknown or non existent values for some entities,
for example, if we haven't made an inventory of some products yet, we don't
want to put a random number in their `stock` attribute. The solution is to use
a `NULL` value, so, the following database is contemplated by our model also:

    
    
    <Gabriel García Marquez, 100, 10/10/2010, 100>
    <Antonio Machado, 100, 20/4/2012, 150>
    <Jorge Luis Borges, 100, 22/6/2014>
    
    <La mala Hora, 1, 20, 100>
    <Antología Poética, 2, 15, NULL>
    <El Aleph, 3, 30, 10>
    

Depending on the semantics, a constraint of can-be-null or can't-be-null can
be added, using some meta language (such as UML) or in plain language:
`Product.name can be null`.

For every attribute there is a list of acceptable values they can hold. That
information is also explained in UML or some other annotation. This is similar
to the concept of programming language types. For example, in our `Product`
entity it makes little sense to say that the `name` attribute contains a
number, or that the `stock` attribute stores a string of characters. Further
constraints are also usual, such as saying: `stock` is a non-negative integer.

Some attributes hold only one value (single-valued attributes) while others
can hold several (multi-valued attributes). The attributes used as examples so
far are all single-valued; for example a person may speak more than one
language, so the language attribute could be multi-valued.

The model contemplates the use of complex attributes which can be broken up
into other atomic attributes (the ones shown before are all atomic
attributes), and are represented as:

    
    
             (street, department, number)
             |
             |                   +--------+ 
    (owner, addres, price)-------|BUILDING|
                                 +--------+
    

These attributes are together because they are semanticaly related. I haven't
used this types of attributes much, so I'm not going to write much about them;
if I ever find them useful, I'll add more information.

## Relationships

Most interesting databases contain not only entities and their attributes but
also relationships between them: an employee employer relationship, a student
professor relationship, and so on. A relationship means several concepts in
real life join together in some way and that union is an important piece of
information.

Let's say we need to model an employee and company relationship. A reasonable
model could be:

    
    
      (name, salary, hire_date, _number_)
      |
    +--------+
    |EMPLOYEE|
    +--------+
    
    
      (name, _id_)
      |
    +-------+
    |COMPANY|
    +-------+
    

But we are missing the connection between them, we can't know what `EMPLOYEE`
works on what `COMPANY`. A refinment:

    
    
      (name, salary, hire_date, _company_, _number_)
      |
    +--------+
    |EMPLOYEE|
    +--------+
    
    
      (name, _id_)
      |
    +-------+
    |COMPANY|
    +-------+
    

This is better, now we can answer important questions such as where does this
employee works?, or what employees does this company has?. But a company is an
entity, not an attribute; it's too important in our model. The way we
represent this relationship is:

    
    
      (name, salary, hire_date, _number_)
      |
    +--------+
    |EMPLOYEE|
    +--------+
        |N
        |
        /\works in
        \/
        |
        |1
    +-------+
    |COMPANY|------(name, _id_)
    +-------+
    

There are more subtleties to this but I wanted to present an example of why
relationships are an important part of the ER model.

What we model in ER as relationships are a mathematical relation between
entity types, meanning that a relationship `R` between entities `E1,...,En` is
a set of `r_i = (e1,...,en)` where `ei` is an entity in the entity set `Ei`.
In the previous example, the model admits a situation like:

    
    
    works_in = {
      (<Jorge Luis Borges, 100, 22/6/2014, 20>, <La casa de Asterión, 120>),
      (<George Orwell, 90, 10/10/2010, 100>, <Ministry of Truth, 121>)
    }
    

In this database, the relationship `works_in`, defined between the entity
types `EMPLOYEE` and `COMPANY`, contains only two elements. This relationship
is said to be **binary** because it involves two entity types; the amount of
entity types participating in a relationships is called it's **degree**.

The definition doesn't say `Ei != Ej` for `0 < i,j <= n`, so a relationship
between the same entity type is possible and they are called **recursive**. A
classical example is the `works for` relationship, in which two `EMPLOYEES`
are related to each other:

    
    
      (name, salary, hire_date, _number_)
      |
    +--------+
    |EMPLOYEE|-------+
    +--------+       |
        |            |
        |            |
        /\works for  |
        \/           |
        |            |
        +------------+
    
    works_for = {
      (<Isidro Parodi, 5, 22/6/1970, 23>, <Jorge Luis Borges, 100, 22/6/2014, 20>),
      (<Winston Smith, 90, 10/10/2010, 100>, <George Orwell, 90, 10/10/2010, 101>)
    }
    

In the example of the `EMPLOYEE`, `COMPANY` and `works_in` relationship I
added a N and a 1 to the lines representing the relationship, they are it's
**cardinality**. The example again:

    
    
      (name, salary, hire_date, _number_)
      |
    +--------+
    |EMPLOYEE|
    +--------+
        |N
        |
        /\works in
        \/
        |
        |1
    +-------+
    |COMPANY|------(name, _id_)
    +-------+
    

In this case the cardinality says that an `EMPLOYEE` can be assosiated with
only one `COMPANY` and a company can have N `EMPLOYEES`. So, for example, the
following case is not allowed by the model:

    
    
    works_in = {
      (<Jorge Luis Borges, 100, 22/6/2014, 20>, <La casa de Asterión, 120>),
      (<Jorge Luis Borges, 300, 22/6/2014, 20>, <Ministry of Truth, 121>)
    }
    

because the same `EMPLOYEE` entity is related to two `COMPANY`ies.

A binary relationship can be 1:1, 1:N, and N:M. A 1:1 relationships means that
an entity of either entity type can only participate once, so

    
    
      (name, salary, hire_date, _number_)
      |
    +-------+
    |MANAGER|
    +-------+
        |1
        |
        /\manages
        \/
        |
        |1
    +------+
    |BRANCH|------(address, _id_)
    +------+
    

admits this

    
    
    manages = {
      (<Jorge Luis Borges, 100, 22/6/2014, 20>, <La casa de Asterión, 120>),
      (<Winston Smith, 90, 10/10/2010, 100>, <Ministry of Truth, 121>)
    }
    

but not this

    
    
    manages = {
      (<Jorge Luis Borges, 100, 22/6/2014, 20>, <La casa de Asterión, 120>),
      (<Winston Smith, 90, 10/10/2010, 100>, <La casa de Asterión, 120>)
    }
    

nor this

    
    
    manages = {
      (<Jorge Luis Borges, 100, 22/6/2014, 20>, <La casa de Asterión, 120>),
      (<Jorge Luis Borges, 100, 22/6/2014, 20>, <Ministry of Truth, 121>)
    }
    

A cardinality of M:N means that every entity can appear several times, is the
default cardinality in the diagram, so, no numbers implies the relationship is
M:N. An example could be

    
    
      (author, _ISBN_, name, _id_)
      |
    +----+
    |BOOK|
    +----+
        |
        |
        /\in
        \/
        |
        |
    +-------+
    |LIBRARY|-----(adress, name, _id_)
    +-------+
    

A `BOOK` can be in a number of `LIBRARY` and viceversa.

The **participation** of an entity i na relationship can be either **partial**
or **total**. A partial participation means that not every entity of an entity
type has to be in the relationship and a total participation means the
opposite.

    
    
      (name, time, credits, _id_)
      |
    +-----+
    |CLASS|
    +-----+
        |1
        |
        /\teaches
        \/
        |
       0|1
    +-------+
    |TEACHER|-----(name, age, gender, _id_)
    +-------+
    

Here, the 0 in the `TEACHER` and the relationship line means the participation
is partial which means not every entity of type `TEACHER` has to be teaching a
`CLASS`. On the other hand, the `CLASS` involvement in the `teaches`
relationship is total, so every `CLASS` has to have an assosiated `TEACHER`.
In other words, under the previous model, no entity of type `CLASS` can exist
in the database if it's not associated with a `TEACHER` entity but a `TEACHER`
can exist who is not teaching a class.

Sometimes the relationships themselves can have their own attributes. This is
useful when we need to store some data which doesn't really belong to any of
the entities participating but in the relationship between them. The notation
is

    
    
    (_email_, username, age, gender)
       |
    +----+
    |USER|--------------+
    +----+              |
                        /\permissions
                        \/-------------(read, write)
    +--------+          |
    |DOCUMENT|----------+
    +--------+
       |
    (_path_, name, creation_date, last_mod)
    

This means that every relationship between a `USER` and a `DOCUMENT` stores
the attributes `read` and `write`. The attributes in the relationships only
make sense in the N:M cardinality; in 1:N relationships the attributes are
better placed in the entity type with N cardinality and the 1:1 relationships
could have their attributes in either.

Even though relationships were defined using n entity types I only used up
until n=3. They are called **ternary** relationships and they allow
cardinality and participation constraints. When an entity type has a
cardinality in a ternary relationship it refers to pairs of the other
participationg entities.

An example will clarify things. Let's say we are building a system in which we
have a set of questions created by us, users and answers. A possible model
could be:

    
    
    (name, _id_)
       |
    +-----+
    |HUMAN|---------------+
    +-----+N              |
                          |
                          |
    (name, _id_)          |
       |                  |
    +----+                /\role_in_project
    |ROLE|----------------\/
    +----+1                |
                           |
                           |
    +-------+0             |
    |PROJECT|--------------+
    +-------+M
           |
    (_name_, _id_)
    

This model says that for every `HUMAN` and `PROJECT` pair in the relationship
only one `ROLE` entity is allowed, so this example is not be consistent with
the model:

    
    
    role_in_project = { 
      (<Donald Knuth, 1>, <TeX, 1>, <Programmer, 1>),
      (<Donald Knuth, 1>, <TeX, 1>, <Web Master, 2>),
      (<Rob Pyke>, <Go, 2>, <Programmer, 2>)
    }
    

