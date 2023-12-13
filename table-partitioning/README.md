# PostgreSQL-playground

# Table partitioning

Table partitioning serves as a strategic database design approach employed to fragment an extensive table into more digestible segments known as partitions. Each partition functions akin to an independent table, housing a distinct subset of the original data. This methodology proves instrumental in enhancing query performance and facilitating more efficient data management, particularly for substantial datasets.


## Types of Partitions

 * Range Partitioning
 * List Partitioning
 * Hash Partitioning

### Range Partitioning

Range partitioning categorizes data into partitions based on a defined range of values within a specific column. This approach proves valuable, especially with time-series data or any dataset exhibiting a natural order. Each partition corresponds to a defined value range, storing data falling within that range. This technique enables efficient data retrieval within specified ranges, ultimately enhancing query performance.

To initiate the process, we generate the parent table named "transaction" by employing the following SQL code:

```
CREATE TABLE transactions (
    transaction_id      SERIAL,
    transaction_date    DATE    NOT NULL,
    product_id          INT,
    quantity            INT,
    amount              NUMERIC
) partition by range (transaction_date);
```
```
docker exec -it database psql -h localhost -U postgres -p 5432 -d main_database -c "CREATE TABLE transactions (
    transaction_id      SERIAL,
    transaction_date    DATE    NOT NULL,
    product_id          INT,
    quantity            INT,
    amount              NUMERIC
) partition by range (transaction_date);"
```

To establish a range-partitioned table for the transactions data, organized according to the transaction_date column, the following steps must be adhered to:


We will generate separate tables to depict each partition, with each table encompassing a specific range of dates.

```
CREATE TABLE transactions_september PARTITION OF transactions
    FOR VALUES FROM ('2023-09-01') TO ('2023-10-01');

CREATE TABLE transactions_october PARTITION OF transactions
    FOR VALUES FROM ('2023-10-01') TO ('2023-11-01');

CREATE TABLE transactions_november PARTITION OF transactions
    FOR VALUES FROM ('2023-11-01') TO ('2023-12-01');
```

```
docker exec -it database psql -h localhost -U postgres -p 5432 -d main_database -c "
CREATE TABLE transactions_september PARTITION OF transactions
    FOR VALUES FROM ('2023-09-01') TO ('2023-10-01');

CREATE TABLE transactions_october PARTITION OF transactions
    FOR VALUES FROM ('2023-10-01') TO ('2023-11-01');

CREATE TABLE transactions_november PARTITION OF transactions
    FOR VALUES FROM ('2023-11-01') TO ('2023-12-01');
"
```

It's crucial to establish constraints on each partition to guarantee accurate data routing to the designated partition. In this instance, we will implement CHECK constraints on the transaction_date column for each partition:

```
ALTER TABLE transactions_september ADD CONSTRAINT transactions_september_check
    CHECK (transaction_date >= '2023-09-01' AND transaction_date < '2023-10-01');

ALTER TABLE transactions_october ADD CONSTRAINT transactions_october_check
    CHECK (transaction_date >= '2023-10-01' AND transaction_date < '2023-11-01');

ALTER TABLE transactions_november ADD CONSTRAINT transactions_november_check
    CHECK (transaction_date >= '2023-11-01' AND transaction_date < '2023-12-01');
```

```
docker exec -it database psql -h localhost -U postgres -p 5432 -d main_database -c "
ALTER TABLE transactions_september ADD CONSTRAINT transactions_september_check
    CHECK (transaction_date >= '2023-09-01' AND transaction_date < '2023-10-01');

ALTER TABLE transactions_october ADD CONSTRAINT transactions_october_check
    CHECK (transaction_date >= '2023-10-01' AND transaction_date < '2023-11-01');

ALTER TABLE transactions_november ADD CONSTRAINT transactions_november_check
    CHECK (transaction_date >= '2023-11-01' AND transaction_date < '2023-12-01');"
```

At this point, we can insert data into the transactions table, and PostgreSQL will autonomously direct the data to the relevant partition based on the transaction_date:
```
INSERT INTO transactions (transaction_date, product_id, quantity, amount)
VALUES ('2023-09-15', 101, 5, 100.00);

INSERT INTO transactions (transaction_date, product_id, quantity, amount)
VALUES ('2023-10-20', 102, 10, 200.00);

INSERT INTO transactions (transaction_date, product_id, quantity, amount)
VALUES ('2023-11-10', 103, 8, 150.00);
```

```
docker exec -it database psql -h localhost -U postgres -p 5432 -d main_database -c "
INSERT INTO transactions (transaction_date, product_id, quantity, amount)
VALUES ('2023-09-15', 101, 5, 100.00);

INSERT INTO transactions (transaction_date, product_id, quantity, amount)
VALUES ('2023-10-20', 102, 10, 200.00);

INSERT INTO transactions (transaction_date, product_id, quantity, amount)
VALUES ('2023-11-10', 103, 8, 150.00);"
```


When querying data, PostgreSQL will automatically access solely the pertinent partitions based on the conditions specified in the WHERE clause.

```
SELECT * FROM transactions;
```


```
docker exec -it database psql -h localhost -U postgres -p 5432 -d main_database -c "SELECT * FROM transactions"
```
```
 transaction_id | transaction_date | product_id | quantity | amount
----------------+------------------+------------+----------+--------
              1 | 2023-09-15       |        101 |        5 | 100.00
              2 | 2023-10-20       |        102 |       10 | 200.00
              3 | 2023-11-10       |        103 |        8 | 150.00
```


```
SELECT * FROM transactions WHERE transaction_date >= '2023-09-01' AND transaction_date < '2023-10-01';

SELECT * FROM transactions WHERE transaction_date >= '2023-10-01' AND transaction_date < '2023-11-01';

SELECT * FROM transactions WHERE transaction_date >= '2023-11-01' AND transaction_date < '2023-12-01';
```


```
docker exec -it database psql -h localhost -U postgres -p 5432 -d main_database -c "
SELECT * FROM transactions WHERE transaction_date >= '2023-09-01' AND transaction_date < '2023-10-01';

SELECT * FROM transactions WHERE transaction_date >= '2023-10-01' AND transaction_date < '2023-11-01';

SELECT * FROM transactions WHERE transaction_date >= '2023-11-01' AND transaction_date < '2023-12-01';"
```

```
 transaction_id | transaction_date | product_id | quantity | amount
----------------+------------------+------------+----------+--------
              1 | 2023-09-15       |        101 |        5 | 100.00
(1 row)

 transaction_id | transaction_date | product_id | quantity | amount
----------------+------------------+------------+----------+--------
              2 | 2023-10-20       |        102 |       10 | 200.00
(1 row)

 transaction_id | transaction_date | product_id | quantity | amount
----------------+------------------+------------+----------+--------
              3 | 2023-11-10       |        103 |        8 | 150.00
```

be more explicit
```
SELECT * FROM transactions_november;
```


```
docker exec -it database psql -h localhost -U postgres -p 5432 -d main_database -c "SELECT * FROM transactions_november"
```
```
 transaction_id | transaction_date | product_id | quantity | amount
----------------+------------------+------------+----------+--------
              3 | 2023-11-10       |        103 |        8 | 150.00
```
