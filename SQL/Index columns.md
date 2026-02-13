When querying a massive table, indexes are mandatory for efficiency.


If you only have read access to indexes, you can find existing ones with a query like this:
```SQL
select a.name, a.index_id, c.name
from sys.indexes as a with (nolock)

left join sys.index_columns as b with (nolock)
on a.object_id=b.object_id and a.index_id=b.index_id

left join sys.all_columns as c with (nolock)
on b.object_id=c.object_id and b.column_id=c.column_id

where a.object_id=123456789
;
```

```Object_ID``` is the table's ID reference. You can find it by searching the ```sys.tables``` table:
```SQL
select top 100 * from sys.tables as c with (nolock) where upper(name) like '%TABLE%';
```
