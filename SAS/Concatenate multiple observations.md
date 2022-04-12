What if you need to put all the values from several observations into a single cell, per some "by" group?

This example starts with 1 record per customer<=>product relationship, and ends with 1 row per customer, with all of their products in a single comma delimmited string.

```SAS
proc sort data=have noduprecs;
by customer;
run;

data want;
  set have;
  by customer;
  
  retain productList; /* verify output - retain is either mandatory or might cause issue - not used this code for a long time */
  format productList $64.;
  length productList $64.;
  
  if first.customer then productList = cats(product);
  else productList = catx(',', productList, product);
  if last.customer then output;
  
  drop product;
  
run;

```
