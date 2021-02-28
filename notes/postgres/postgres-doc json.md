### Plain JSON type

In 2012, PostgreSQL 9.2 introduced the first `JSON` data type in Postgres. It had syntax validation but underneath it stored the incoming document directly as text with white spaces included. It wasn’t very useful for real world querying, index based searching and other functionalities you would normally do with a JSON document.

### JSONB

In late 2014, PostgreSQL 9.4 introduced the `JSONB` data type and most importantly improved the querying efficiency by adding **[indexing](https://blog.arctype.com/database-index/).** 

The `JSONB` data type stores JSON as a binary type. This introduced overhead in processing since there was a conversion involved but it offered the ability to index the data using GIN/Full text based indexing and included additional operators for easy querying.

### JSONPath

With JSON’s increasing popularity, the 2016 SQL Standard brought in a new   standard/path language for navigating JSON data. It’s a powerful way of     searching JSON data very similar to XPath for XML data. PostgreSQL 12 introduced support for the JSON Path standard.

We will see examples of JSON, JSONB, and JSONPath in the sections below. An important thing to note is that all JSON functionality is natively present in the database. There is no need for a contrib module or an external package to be installed.

## JSON Example in Postgres

Lets create a Postgres table to store lunch orders with a JSON data type.

```sql
create table LunchOrders(student_id int, order json);
```

Now we can insert JSON formatted data into our table with an `INSERT` statement. 

```sql
insert into LunchOrders values(100, '{
	"order_date": "2020-12-11",
	"order_details": {
    	"cost": 4.25,
        "entree": ["pizza"],
    	"sides": ["apple", "fries"],
    	"snacks": ["chips"]}
	}'      
);

insert into LunchOrders values(100, '{
	"order_date": "2020-12-12",
	"order_details": {
    	"cost": 4.89,
        "entree": ["hamburger"],
    	"sides": ["apple", "salad"],
    	"snacks": ["cookie"]}
	}'      
);
```

If you do a  `Select *`  from the table, you would see something like below.

![](https://blog.arctype.com/content/images/2021/01/Screen-Shot-2021-01-30-at-10.10.32-PM.png)


Inserting data into a `JSONB` column is exactly the same, except we change the data type to `jsonb`.

```sql
create table LunchOrders(student_id int, orders jsonb);
```

## How to Query JSON Data in Postgres

Querying data from JSON objects uses slightly different operators than the ones that we use for regular data types ( `=`, `<` , `>`, etc). 

Here are some of the most common JSON operators:

Operator

Description

`\->`

Select a key/value pair

`\->>`

Filter query results

`#>`

Selecting a nested object

`#>>`

Filter query results in a nested object

`@>`

Check if an object contains a value

[Full list of JSON operators](https://www.postgresql.org/docs/12/functions-json.html).

The `->` and `->>` operators work with both `JSON` and `JSONB` type data. The rest of the operators are _full text search_ operators and only work with the `JSONB` data type.

Let's see some examples of how to use each operator to query data in our `LunchOrders` table. 

### Getting values from a JSON object

We can use the `->` operation to find every day that a specific student bought a school lunch. 

```sql
select orders -> 'order_date'
from lunchorders
where student_id = 100;
```

![](https://blog.arctype.com/content/images/2021/01/Screen-Shot-2021-01-30-at-10.13.44-PM.png)

### Filtering JSON data using a `WHERE` clause

We can use the `->>` operator to filter for only lunch orders on a specific date.

```sql
select orders
from lunchorders
where orders ->> 'order_date' = '2020-12-11';
```

![](https://blog.arctype.com/content/images/2021/01/Screen-Shot-2021-01-30-at-10.44.10-PM.png)

This query is similar to the `=` operator that we would normally use, except we have to first add a `->>` operator to tell Postgres that the `order_date` field is in the `orders` column.

### Getting data from an array in a JSON object 

Let's say we wanted to find every side dish that a specific student has ordered.

The `sides` field is nested inside the `order_details` object, but we can access it by chaining two `->` operators together.

```sql
select
  orders -> 'order_date',
  orders -> 'order_details' -> 'sides'
from
  lunchorders
where
  student_id = 100;
```

![](https://blog.arctype.com/content/images/2021/01/Screen-Shot-2021-01-30-at-11.05.10-PM.png)

Great now we have arrays of the sides that student 100 ordered each day! What if we only wanted the first side in the array? We can chain together a _third_ `->` operator and give it the **array index** we're looking for.

![](https://blog.arctype.com/content/images/2021/01/Screen-Shot-2021-01-30-at-11.10.14-PM.png)

Read further -> [Citation](https://blog.arctype.com/json-in-postgresql/?utm_campaign=json-postgres&utm_medium=blog&utm_source=reddit) 

#database #architecture 