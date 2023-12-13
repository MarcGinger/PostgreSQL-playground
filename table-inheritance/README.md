# PostgreSQL-playground

# Table inheritance

Table inheritance enables the establishment of a hierarchical connection among tables. This functionality involves the specification of a parent table, from which child tables inherit columns and certain constraints such as CHECK constraints and NOT NULL constraints.

### How it works


To initiate the process, we generate the parent table named "company" by employing the following SQL code:

```
CREATE TABLE company (
  id                SERIAL      PRIMARY KEY,
  name              TEXT        NOT NULL,
  tax_number        TEXT        NOT NULL,
);
```

```
docker exec -it database psql -h localhost -U postgres -p 5432 -d main_database -c "CREATE TABLE company (
  id                SERIAL      PRIMARY KEY,
  name              TEXT        NOT NULL,
  tax_number        TEXT        NOT NULL
);"
```

Following that, we'll establish child tables that derive from the "company" table. Two distinct company categories, namely supplier and employer, will be created. Each child table will possess its unique columns alongside the inherited ones. The establishment of the inheritance relationship is accomplished using the ```INHERITS``` keyword:

```
CREATE TABLE supplier (
  code              TEXT        NOT NULL,
  bank_name         TEXT        NOT NULL,
  account_number    TEXT        NOT NULL,
) INHERITS (company);
 
CREATE TABLE employer (
  code              TEXT        NOT NULL,
  manager           TEXT        NOT NULL,
) INHERITS (company);
```
```
docker exec -it database psql -h localhost -U postgres -p 5432 -d main_database -c "
CREATE TABLE supplier (
  code              TEXT        NOT NULL,
  bank_name         TEXT        NOT NULL,
  account_number    TEXT        NOT NULL
) INHERITS (company);

CREATE TABLE employer (
  code              TEXT        NOT NULL,
  manager           TEXT        NOT NULL
) INHERITS (company);
"
```
check the tables have been created

```
docker exec -it database psql -h localhost -U postgres -p 5432 -d main_database -c "\dt"
```


Subsequently, we insert various rows into the child tables:


```
INSERT INTO supplier
  (name, tax_number, code, bank_name, account_number)
  VALUES ('Supplier Co', 'TN0001', 'CD001', 'Some bank', '7011230000');

INSERT INTO employer
  (name, tax_number, code, manager)
  VALUES ('Supplier Co', 'TN0002', 'CD002', 'Sleepy Joe');
```
```
docker exec -it database psql -h localhost -U postgres -p 5432 -d main_database -c "
INSERT INTO supplier
  (name, tax_number, code, bank_name, account_number)
  VALUES ('Supplier Co', 'TN0001', 'CD001', 'Some bank', '7011230000');
  
INSERT INTO employer
  (name, tax_number, code, manager)
  VALUES ('Employer Co', 'TN0002', 'CD002', 'Sleepy Joe');
"
```

Upon querying each child table separately, the result only includes the rows inserted into that specific table, aligning with expectations. Nevertheless, querying the parent table yields all the companies from both child tables:

```
SELECT * FROM supplier;

 id |    name     | tax_number | code  | bank_name | account_number
----+-------------+------------+-------+-----------+----------------
  1 | Supplier Co | TN0001     | CD001 | Some bank | 7011230000

 ```

 ```
 docker exec -it database psql -h localhost -U postgres -p 5432 -d main_database -c "SELECT * FROM supplier;"
 ```

 ```
SELECT * FROM employer;

 id |    name     | tax_number | code  |  manager
----+-------------+------------+-------+------------
  2 | Employer Co | TN0002     | CD002 | Sleepy Joe

 ```

 ```
 docker exec -it database psql -h localhost -U postgres -p 5432 -d main_database -c "SELECT * FROM employer;"
 ```
 
 ```
SELECT * FROM company;

 id |    name     | tax_number
----+-------------+------------
  1 | Supplier Co | TN0001
  2 | Employer Co | TN0002

 ```

 ```
 docker exec -it database psql -h localhost -U postgres -p 5432 -d main_database -c "SELECT * FROM company;"
 ```
# for the maniacs


A table possesses the capability to inherit from multiple parent tables. In such instances, the table amalgamates the columns outlined by all the parent tables, forming a union of their attributes.

```
CREATE TABLE supplier_employer (
  some_value        TEXT        NOT NULL
) INHERITS (supplier, employer);

```
