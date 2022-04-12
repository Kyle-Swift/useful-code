Iterate through sequential variables with the same name, in order to find out how many there are. The original use case was importing an excel file where previous addresses were posted in new columns instead of rows, and wanting to be able to re-run the code without having to manually look how many columns the sheet had.

```SAS

%macro findUpperBound(ds, var, extraDistinction=);
  
  /*
  ds = input dataset
  var = name of the variable array (without the number, e.g. supply "post_code_", where the columns are like "post_code_1")
  extraDistinction = used to ensure re-runs can be distinct, for example if you're doing the same check on 2 different datasets
  */
  
  %local dsid rc;
  
  %let dsid=%sysfunc(open(&ds.)); /* open the dataset in memory */
  
  %let i=1;
  
  /* varnum() function returns the position of the variable. 0 if missing */
    
  /* if the array doesn't exist, throw an error instead of running */
  %if %sysfunc(varnum(&dsid.,&var.1))=0 %then %do;
    %put ERROR: &var. array doesn't exist! Stopping execution.;
    %Abort;
  %end;
  
  /* now iterate until you find the upper bound */
  %do %while (%sysfunc(varnum(&dsid.,&var.&i.)));
  %let i = %eval(&&i.+1);
  %end;
  
  /* subtract one to get the final positive result - could perhaps be avoided if the %do %while loop was %do %until, instead. */
  %let i = %eval(&&i.-1);
  
  /* post the results to a dataset. method varies depending on whether this is the first run in the current session */
  proc sql;
    
    /* first run of the session (tracking dataset doesn't exist yet) */
    %if %sysfunc(exist(work.counterTable))=0 %then %do;
    create table work.counterTable as
    select distinct "&var." as Variable length=64., &i. as Volume, "&extraDistinction." as Grouping length=64
    from &ds. /* not actually using this dataset. just the query needs a valid FROM statement in order to execute */
    %end;
    
    %else %do; /* subsequent runs post results to the existing dataset */
    insert into work.counterTable
    set Variable = "&var."
      , Volume = &i.
      , Grouping = "&extraDistinction."
    %end;
    
    ;
  quit;
  
  %let rc=%sysfunc(close(&dsid.));  
  
%mend findUpperBound;

%findUpperBound(work.data, array_columns_, textTextText); /* call the macro */

/* you can grab the results into a variable in the usual way, to use going forwards. depending on use case, you may want to */
/* specify the extraDistinction, or perhaps take the max. respectively: */
proc sql;
  select Volume into:upperBoundSpecific from work.counterTable where variable = 'array_columns_' and Grouping='textTextText';
  select max(Volume) into:upperBoundLargest from work.counterTable;
quit;
```
