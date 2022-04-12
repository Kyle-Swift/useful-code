For when you need to perform an action with one or more variables, per row in a dataset. This code inserts the values of each observation into macro variables.
Most common use case would be importing a long list of files, which are themselves listed in a SAS dataset you've already imported, for example.

```SAS
%macro loopThroughVariables(data);

  %let id=%sysfunc(open(&data.));
  %let NObs=%sysfunc(attrn(&id.,NOBS)); /* find the number of observations, so we know how many loops to do */

  %syscall set(id); /* opens the dataset in memory */

  %do i=1 %to &NObs.; /* loop once per row in the source data */
    %let rc=%sysfunc(fetchobs(&id.,&i.)); /* insert observations at the current row into memory */


    /* your code here -- the current row's values are stored as macro variables with the same name as in the original data */
    /* continuing with the mass-import example, if your dataset held "location", "name" and "extension", this would work without further assignment: */
    /* proc import datafile=&location.&name. dbms=&extension.; run; */

  %end;

  %let id=%sysfunc(close(&id.));

%mend loopThroughVariables;

%loopThroughVariables(work.dataset); /* run the macro for your desired file */
```
