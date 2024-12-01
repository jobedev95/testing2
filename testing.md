# Innan all optimering:
| id   | select_type | table | type   | possible_keys                    | key           | key_len | ref                          | rows | r_rows | filtered | r_filtered | Extra                                        |
|------|-------------|-------|--------|----------------------------------|---------------|---------|------------------------------|------|--------|----------|------------|----------------------------------------------|
|    1 | SIMPLE      | os    | ALL    | PRIMARY                          | NULL          | NULL    | NULL                         | 10   | 10.00  |   100.00 |      10.00 | Using where; Using temporary; Using filesort |
|    1 | SIMPLE      | o     | ref    | PRIMARY,CustomerID,OrderStatusID | OrderStatusID | 4       | ecommercedb.os.OrderStatusID | 1    | 13.00  |   100.00 |      46.15 | Using where                                  |
|    1 | SIMPLE      | c     | eq_ref | PRIMARY                          | PRIMARY       | 4       | ecommercedb.o.CustomerID     | 1    | 1.00   |   100.00 |     100.00 |                                              |
|    1 | SIMPLE      | od    | ref    | OrderID,ProductID                | OrderID       | 4       | ecommercedb.o.OrderID        | 1    | 11.33  |   100.00 |     100.00 |                                              |
|    1 | SIMPLE      | p     | eq_ref | PRIMARY                          | PRIMARY       | 4       | ecommercedb.od.ProductID     | 1    | 1.00   |   100.00 |     100.00 |                                              |


# Utan skapade index men med optimerad filtrering i WHERE-klausulen:
```bash
+------+-------------+-------+--------+----------------------------------+---------------+---------+--------------------------+------+--------+----------+------------+---------------------------------+
| id   | select_type | table | type   | possible_keys                    | key           | key_len | ref                      | rows | r_rows | filtered | r_filtered | Extra                           |
+------+-------------+-------+--------+----------------------------------+---------------+---------+--------------------------+------+--------+----------+------------+---------------------------------+
|    1 | SIMPLE      | os    | const  | PRIMARY                          | PRIMARY       | 4       | const                    | 1    | NULL   |   100.00 |       NULL | Using temporary; Using filesort |
|    1 | SIMPLE      | o     | ref    | PRIMARY,CustomerID,OrderStatusID | OrderStatusID | 4       | const                    | 13   | 13.00  |   100.00 |      46.15 | Using where                     |
|    1 | SIMPLE      | c     | eq_ref | PRIMARY                          | PRIMARY       | 4       | ecommercedb.o.CustomerID | 1    | 1.00   |   100.00 |     100.00 |                                 |
|    1 | SIMPLE      | od    | ref    | OrderID,ProductID                | OrderID       | 4       | ecommercedb.o.OrderID    | 1    | 11.33  |   100.00 |     100.00 |                                 |
|    1 | SIMPLE      | p     | eq_ref | PRIMARY                          | PRIMARY       | 4       | ecommercedb.od.ProductID | 1    | 1.00   |   100.00 |     100.00 |                                 |
+------+-------------+-------+--------+----------------------------------+---------------+---------+--------------------------+------+--------+----------+------------+---------------------------------+
```


# Med optimerad filtrering + sammansatt index på OrderStatusID och OrderDate.
`CREATE INDEX index_status_date ON Orders (OrderStatusID, OrderDate)`
```bash
+------+-------------+-------+--------+------------------------------------+-----------------+---------+--------------------------+------+--------+----------+------------+---------------------------------+
| id   | select_type | table | type   | possible_keys                      | key             | key_len | ref                      | rows | r_rows | filtered | r_filtered | Extra                           |
+------+-------------+-------+--------+------------------------------------+-----------------+---------+--------------------------+------+--------+----------+------------+---------------------------------+
|    1 | SIMPLE      | os    | const  | PRIMARY                            | PRIMARY         | 4       | const                    | 1    | NULL   |   100.00 |       NULL | Using temporary; Using filesort |
|    1 | SIMPLE      | o     | range  | PRIMARY,CustomerID,idx_status_date | idx_status_date | 7       | NULL                     | 6    | 6.00   |   100.00 |     100.00 | Using index condition           |
|    1 | SIMPLE      | c     | eq_ref | PRIMARY                            | PRIMARY         | 4       | ecommercedb.o.CustomerID | 1    | 1.00   |   100.00 |     100.00 |                                 |
|    1 | SIMPLE      | od    | ref    | OrderID,ProductID                  | OrderID         | 4       | ecommercedb.o.OrderID    | 1    | 11.33  |   100.00 |     100.00 |                                 |
|    1 | SIMPLE      | p     | eq_ref | PRIMARY                            | PRIMARY         | 4       | ecommercedb.od.ProductID | 1    | 1.00   |   100.00 |     100.00 |                                 |
+------+-------------+-------+--------+------------------------------------+-----------------+---------+--------------------------+------+--------+----------+------------+---------------------------------+
```


# Med optimerad filtrering + sammansatt index + index på TotalPrice i fallande ordning.
`CREATE INDEX index_total_price ON OrderDetails (TotalPrice DESC)`
```bash
+------+-------------+-------+--------+------------------------------------+-----------------+---------+--------------------------+------+--------+----------+------------+---------------------------------+
| id   | select_type | table | type   | possible_keys                      | key             | key_len | ref                      | rows | r_rows | filtered | r_filtered | Extra                           |
+------+-------------+-------+--------+------------------------------------+-----------------+---------+--------------------------+------+--------+----------+------------+---------------------------------+
|    1 | SIMPLE      | os    | const  | PRIMARY                            | PRIMARY         | 4       | const                    | 1    | NULL   |   100.00 |       NULL | Using temporary; Using filesort |
|    1 | SIMPLE      | o     | range  | PRIMARY,CustomerID,idx_status_date | idx_status_date | 7       | NULL                     | 6    | 6.00   |   100.00 |     100.00 | Using index condition           |
|    1 | SIMPLE      | c     | eq_ref | PRIMARY                            | PRIMARY         | 4       | ecommercedb.o.CustomerID | 1    | 1.00   |   100.00 |     100.00 |                                 |
|    1 | SIMPLE      | od    | ref    | OrderID,ProductID                  | OrderID         | 4       | ecommercedb.o.OrderID    | 10   | 11.33  |   100.00 |     100.00 |                                 |
|    1 | SIMPLE      | p     | eq_ref | PRIMARY                            | PRIMARY         | 4       | ecommercedb.od.ProductID | 1    | 1.00   |   100.00 |     100.00 |                                 |
+------+-------------+-------+--------+------------------------------------+-----------------+---------+--------------------------+------+--------+----------+------------+---------------------------------+
```

