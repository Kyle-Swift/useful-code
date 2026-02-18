I find Windows functions very hard to write from memory. This is quite niche and simplistic but I hope by documenting this, I may train myself to not need to Google it every time.


```select row_number() over (partition by ___ order by ___)``` is used to add a row number at a grouped level. Each new group restarts the count at 1.

____________
____________

1)

If you have a Heap of customer detail records and need to identify their sequence, so that you can pick ```seq=1``` to get the latest data, this is a very simple way to do so:
```SQL
select a.Customer
  , b.* /* Table 'b' is our heap. Pick whatever data attributes you need. */
  , row_number() over (partition by a.Customer order by b.Record_Created_Date desc) as Seq
```
This is better practice than making analysts themselves pick the latest record, because selecting the latest record without the sequence column is not entirely user friendly, and looks something like:
```SQL
Group by Customer, data, data, data, data /* All the columns you want... */
having Record_Created_Date=max(Record_Created_Date)
```

____________
____________

2)

In the second example, I want to order student test results by subject area and overall marks, which could be used to identify trends, such as how specific students react to specific material.

Obviously you can just use "order by" instead of the windows function, in this case.

We're assuming that the ranked column has some downstream use, such as giving praise to a struggling student who did really well in one subject area.

```SQL
select a.Student
  , a.Total_Score_outOfThousand
  , b.Subject
  , b.Subject_Score_outOfHundred
  , row_number() over (partition by b.Subject
                    order by b.Subject_Score_outOfHundred desc, a.Total_Score_outOfThousand asc) as RN
                    /* Sorted by top marks in the subject but lowest overall grade. */
```


