# Normalization

A multitude of database designs can model a real world problem correctly. Some
of those designs are faster than others. Some are easier to understand as a
programmer. A database that contains a single table to manage a lot of real
world entities is probably too akward to use: data is probably duplicated,
redundant and hard to update and delete.

[Normalization](https://wikipedia.org/wiki/Database_normalization) is a way to
measure how good a database design is, a set of rules and processes by which a
programmer can improve their database schema. A normalized database is easier
to understand, has less redundancy and it's easier to use.

Some of the pitfalls of database design are:

  * **Table semantics** : what does a table represent in the problem domain? what each of the attributes model?
  * **Data redudancy** : saving the same piece of data several times wastes space and makes the database cumbersome to use and change
  * **NULL values** : putting too many columns which accept NULL values obscures the semantics of the tables and wastes space
  * **Update anomalies** : adding, deleting and updating rows is too complicated or error prone

If the semantics of the tables in your database schemas are clear it means
that the mapping between the reality and your model is clear, so an
understanding of the problem can translate neatly to an understanding of the
database. To make the meaning of a table clear means to have good table names
and good attribute names, just as function and variable names should be clear
to aid other programmers reading our code. The types of the attributes is also
important, for example, if a column holds timestamps, a date or time type is
probably more appropiate than a string.

A relational database stores data and relates them to each other. The more
redundancy, the more space a database needs. Of course for small databases
this is not a big problem, but as the database grows it becomes more
inconvinient. The more redundant a database is, the harder to maintain it's
integrity and make changes to it becomes.

Having a table with too many `NULL`s complicates operations such as `joins` or
`select`s and aggregation functions such as `count`. A `null` value in a
column can mean that there is no acceptable value, that is not known or
missing, so the semantics of the attributes can be muddled. Also, avoidable
`null` values can waste a lot of space. A way to solve it is moving that
column into a new table which links to the original one, for example moving

    
    
    Human(_id_, name, phone)
    

into

    
    
    Human(_human_id_, name)
    Phone(_phone_id_, *human_id, number)
    

One of the most important problems that normalization solves is the update
anomalies.

## Update Anomalies

The update anomalies happen when a bad design makes it complicated to update,
insert or delete data. It's very related to the redundancy problem.

There are three types of anomalies:

  * **Insertion anomalies**
  * **Deletion anomalies**
  * **Modification anomalies**

### Insertion Anomalies

A table has insertion anomalies when adding a new row with certain attributes
requires the existence of other attributes. It's produced when there's an
artificial dependency between attributes.

For example, in the table

    
    
    Book(_id_, editorial_name, editorial_adress, title, author, price)
    

it's not possible to just insert a new editorial, represented by the
attributes `editorial_name` and `editorial_adress`, without adding a new
`title`, `author` and `price`. To fix it you can split the table in two:

    
    
    Editorial(_name_, address)
    Book(_id_, title, author, price, *editorial)
    

### Delete Anomalies

A delete anomaly happens when deleting some data in a table can produce the
unintended side effect of deleting other data from the database. It's produced
by a bad design in the database in which a table includes some attributes it
shouldn't. For example, when an entity is modeled by a set of attributes
within a table for another identity.

    
    
    Company(department_name, department_building, _employee_id_, employee_salary)
    

In the `Company` table above, a department is modeled as the two attributes
`department_name` and `department_building`. It's possible that by deleting an
employee, for example the last employee in a department, a department
disappears. To remove that delete anomaly, the `Company` relation could be
broken into two tables, one modeling departments and one modeling employees:

    
    
    Employee(_employee_id_, employee_salary, *department_name)
    Department(_department_name_, department_building)
    

### Modification Anomalies

This anomaly makes changing tuples annoying. If the table has a bad design,
changing one row forces the programmer to change other rows for the sake of
mantaining data integrity. It is clear here that redundant information is a
big problem: all of those redundant copies need to be updated as well.

    
    
    Company(department_name, department_building, _employee_id_, employee_salary)
    

If the department changes it's name, every row in the `Company` table
containing that department needs to be changed. The same fix works here:

    
    
    Employee(_employee_id_, employee_salary, *department_name)
    Department(_department_name_, department_building)
    

In this design, if a department name changes, only one change is needed.

One way to avoid most of these problems is to use an ER model to create the
design and then transform it into a relational model; that way entities are
matched to tables and redundancy is greatly reduced. It's not imposible for
database design errors to appear while following the ER to MR transformation,
but it reduces the probability.

## Normalization Process

The normalization process is about detecting dependencies between attributes
in tables in a scheme and split those tables into new ones arranging the
attributes in ways that reduce or eliminate the update anomalies and data
redundancy.

Those dependencies are formally defined as [functional
dependencies](https://wikipedia.org/wiki/Functional_dependency) and are set by
the programmer using real world knowledge about the entities modeled.

### Some definitions

Given two attributes `X` and `Y` in a schema `R` a functional dependency
between `X` and `Y`, denoted as `X->Y`, means that for every possible relation
set `r` (set of tuples in `R`) and two tuples `t1` and `t2` in `r` then

    
    
    t1[X]=t2[X] => t1[Y]=t2[Y]
    

When that property holds it's said that `Y` is functionally dependent on `X`,
or that `X` determines `Y`.

Functional dependencies are not derived from a particular `r` but from
knowledge of the real world thing beign modeled, a property of the thing.

A functional dependency between two attributes `X->Y` means that, in a way,
the set of attributes `X` functions as a key for other attributes `Y`.

If `X->R` then `X` is a [superkey](https://wikipedia.org/wiki/Superkey). If
there is no `Z` in `X` which `Z->R`, then `X` is a key. The set of all keys is
known as a schema's [candidate
keys](https://wikipedia.org/wiki/Candidate_key). A [primary
key](https://wikipedia.org/wiki/Primary_key) is chosen among the candidate
keys.

A prime attribute in a schema `R` is an attribute which is part of some
candidate key.

A functional dependency `X->Y` in a schema `R` is a transitive dependency if
there exist a set of attributes `Z` in `R` such that `X->Z`, `Z->Y` and `X` is
not determined by `Z`.

A functional dependency `X->Y` is parcial if there is a `Z` in `X` that
`Z->Y`. Oherwise, the dependency is said to be full.

## Normal Forms

A [normal form](https://wikipedia.org/wiki/Database_normalization#Normal_form)
is a property of a table that increases the probability of it beign a good
design. In the normalization process, a table which is not in a normal form,
gets separated into several tables conforming to that form. They live in a
hierarchy in which one extends the previous one. They help in getting rid of
the modification anomalies.

The normal forms are:

  * [1NF](https://wikipedia.org/wiki/First_normal_form)
  * [2NF](https://wikipedia.org/wiki/Second_normal_form)
  * [3NF](https://wikipedia.org/wiki/Third_normal_form)
  * [BCNF](https://wikipedia.org/wiki/Boyce%E2%80%93Codd_normal_form)

### 1NF: first normal form

A table is in 1NF if all it's attributes are atomic. This means that this form
doesn't allow multivalues or composite attributes.

Usually, modern database tables are already in 1NF.

### 2NF: second normal form

A schema `R` is in 2NF if it's in 1NF and no non-prime attribute is partially
dependent on any key.

A non-prime attribute can be thought of as providing information but not
identity. If a non-prime attribute is partially functionally dependent on a
key, then they are storing data about an entity which can be identify with
part of said key. Then, we can think that that key can be split into several
sets and separating them into tables along with the other attributes.

Let's take the following schema:

    
    
    R(A1, A2, A3, A4, A5, A6)
    

with the functional dependencies generated by:

    
    
    {A1, A2}->A3, A1->A, A2->A5, A2->A6
    

The non-prime attributes are `A3` to `A6`. It's primary keys are `{A1, A2}`.
`R` does not conform with the 2NF because `A4` it's partialy functionally
dependent with the primary key and `A5` is also.

`R` can be refined to comply with the 2NF by separating into:

    
    
    R1(_A1_, _A2_, A3)
    R2(_A1_, A4)
    R3(_A2_, A5, A6)
    

### 3NF: third normal form

A relation schema `R` is in 3NF if it's in 2NF and for any non-trivial `X->A`
on `R` then one of the following is true:

  * `X` is a superkey of `R`
  * `A` is a prime attribute

A non trivial dependency `X->A` is one in which `A` is not in `X`.

For example, `R(_K_, A1, A2, A3)` with functional dependencies `K->A1`,
`K->A2`, `A2->A3` and the candidate keys `{K}`. `R` is not in 3NF because in
`A2->A3`, `A2` is not a superkey. To solve it, we can produce the following
relations:

    
    
    R1(_K_, A1, A2)
    R2(_A2_, A3)
    

### BCNF: Boyce-Codd normal form

A relation schema `R` is in BCNF if for all non-trivial functional dependency
`X->Y` in `R`, `X` has to be a superkey.

For example, in the relation `R(K, A1, A2, A3)` with functional dependencies
`K->{A1, A2, A3}`, `{A1, A2}->{K, A1}` and `A3->A1`. Since `A3` is not a
superkey, then `R` is not in `BCNF`. A possible solution is:

    
    
    R1(_K_, A2, A3)
    R2(_A3_, A1)
    

