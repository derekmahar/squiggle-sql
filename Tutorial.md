This short tutorial will take you through the basics of using Squiggle.

## Creating a basic SELECT query ##

The core of Squiggle revolves around the SelectQuery class. When creating a new SELECT statement, you instantiate a new SelectQuery and add query attributes to it.

```
SelectQuery select = new SelectQuery();
```

Instances of Table are used to represent each table participating in a query. Table takes a name as a parameter to the constructor, which is the name of the table in the database.

Now columns can be added to the query using SelectQuery.addColumn(). This takes two parameters; the Table containing the column and the name of the column in the database.

```
select.addColumn(orders, "id");
select.addColumn(orders, "total_price");
```

The generated SQL can be obtained using SelectQuery.toString().

```
System.out.println(select.toString());
```

Which outputs:

```
SELECT
   orders_table.id ,
   orders_table.total_price
FROM
   orders_table
```

## Adding WHERE criteria ##

The SelectQuery.addCriteria() method can be used to add criteria to the WHERE clause of a query, which takes a Criteria object as a parameter.

Criteria itself is an abstract class as there are many different types of criteria that can be appended to a WHERE clause.

MatchCriteria is used for specifying a simple match such as `MY_COL = 'foo'` or `C <= 6`.

```
    select.addCriteria(new MatchCriteria(orders, "status", MatchCriteria.EQUALS, "processed"));
    select.addCriteria(new MatchCriteria(orders, "items", MatchCriteria.LESS, 5));
```

The parameters for MatchCriteria constructor are:

  * Table table: The table containing the column used in the match.
  * String columnName: The database column name in the table.
  * String matchType: The operator to use. MatchCriteria provides some constants for these.
  * String/int/float/boolean value: The value to match against. There are multiple overloaded constructors for different types.

The query now looks like:

```
SELECT
   orders_table.id ,
   orders_table.total_price
FROM
    orders_table
WHERE
    orders_table.status = 'processed' AND
    orders_table.items
```

Another type of Criteria is the InCriteria, which is used for specifying MY\_COL IN ('a','b','c',....) criteria. The values can be specified using an array or a collection.

```
select.addCriteria(new InCriteria(orders, "delivery", new String[] { "post", "fedex", "goat" } ));
```

Which produces:
```
SELECT
    orders_table.id ,
    orders_table.total_price
FROM
    orders_table
WHERE
    orders_table.status = 'processed' AND
    orders_table.items orders_table.delivery IN (
        'post','fedex','goat'
    )
```

Many other types of criteria are defined in the com.truemesh.squiggle.criteria package.

## Joining tables ##

To add a new table to the query using a join, two Tables must be instantiated and then passed to the SelectQuery.addJoin() method.

The parameters for SelectQuery.addJoin() are:

  * Table sourceTable
  * String sourceColumn
  * Table destTable
  * String destColumn

So, this:

```
Table warehouses = new Table("warehouses_table");
select.addJoin(orders, "warehouse_id", warehouses, "id");
```

Modifies the SELECT statement to insert the SQL necessary to include the join:

```
SELECT
    orders_table.id ,
    orders_table.total_price
FROM
    orders_table ,
    warehouses_table
WHERE
    orders_table.status = 'processed' AND
    orders_table.items < 5 AND
    orders_table.warehouse_id = warehouses_table.id
```

Of course, joining the table is pretty useless on its own, so you can also use the table in the query in the same way you could use the first table.

```
select.addColumn(warehouses, "location");
select.addCriteria(new MatchCriteria(warehouses, "size", MatchCriteria.EQUALS, "big"));
```

Which adds a new column to the selection and a new match criteria:

```
SELECT
    orders_table.id ,
    orders_table.total_price ,
    warehouses_table.location
FROM
    orders_table ,
    warehouses_table
WHERE
    orders_table.status = 'processed' AND
    orders_table.items < 5 AND
    orders_table.warehouse_id = warehouses_table.id AND
    warehouses_table.size = 'big'
```

SelectQuery.addJoin() is a bit of convenient syntactic sugar: it adds a MatchCriteria that refers to columns of two different tables.  You can add a MatchCriteria directly if its easier.  The SelectQuery will still add the appropriate tables to the FROM clause when it generates SQL.

## Performing sub-selects ##

To perform a sub-select (query within query), you build a separate query using a new instance of SelectQuery, and then add it to the first using an InCriteria.

So, here's a new select statement:

```
SelectQuery subSelect = new SelectQuery();
Table offers = new Table("offers_table");
subSelect.addColumn(offers, "location");
subSelect.addCriteria(new MatchCriteria(offers, "valid", MatchCriteria.EQUALS, true));
```

Which, on its own, produces:

```
SELECT
    offers_table.location
FROM
    offers_table
WHERE
    offers_table.valid = true
```

This can then be used as a sub-select in the original query by adding the new SelectQuery as a parameter to InCriteria:

```
select.addCriteria(new InCriteria(warehouses, "location", subSelect));
```

Which leaves us with:

```
SELECT
    orders_table.id ,
    orders_table.total_price ,
    warehouses_table.location
FROM
    orders_table ,
    warehouses_table
WHERE
    orders_table.status = 'processed' AND
    orders_table.items warehouses_table.location IN (
        SELECT
            offers_table.location
        FROM
            offers_table
        WHERE
            offers_table.valid = true
    )
```

Phew!

## Next steps ##

This concludes the two minute tutorial. Maybe it took a bit longer, but hopefully not that much.

You should have a solid idea of how to use Squiggle now, including:

  * Building SELECT statements.
  * Adding WHERE criteria.
  * Joining tables.
  * Performing sub-selects.

From here, the best way to learn more about Squiggle is by looking at the [examples](http://code.google.com/p/squiggle-sql/source/browse/#svn/trunk/tests/com/truemesh/squiggle/tests), exploring the API, looking at the source code and experimenting.