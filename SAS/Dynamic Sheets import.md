This code offers a way to import all data sheets from various datafiles, without needing to know all of their names.
My use case is an excel file containing multiple sheets with identical columns.
Assumes linux filepath rather than windows (forward slashes instead of back), and may only work for xlsx in current state.

```SAS
%macro importAllSheets(fullFilePath);
  
  /* first goal is to split the full file path into component parts: path, filename, and extension (e.g. 'xlsx') */
  /* 1) everything after the final '/' is the filename */
  %let fileName=%sysfunc(scan(&fullFilePath., -1, '/')); 
  /* 2) everything up to the final '/' is the path */
  %let fileFolder=%substr(&fullFilePath., 1, (%length(&fullFilePath.)-%length(fileName)-1)));
  /* 3) everything after the final '.' is the extension. upcase is not usually necessary, depending on import method */
  %let fileType=%sysfunc(upcase(%sysfunc(scan(&fullFilePath., -1, '.')));
  
  /* now grab all sheetnames */
  libname fileLib &fileType. "&fullFilePath.";
  proc contents data=fileLib._all_ memtype=data out=work.procContentsOutput noprint; run;
  proc sql; select distinct memname into:sheetNames separated by '|' from work.procContentsOutput; quit;
  libname fileLib CLEAR;
  
  /* proc import for every sheet */
  %do sheetNumber=1 %to %sysfunc(countw(&sheetNames., '|'));
    
    /* isolate the next sheet name */
    %let sheet=%sysfunc(scan(&sheetNames.,&sheetNumber,'|'));
    
    /* basic... adjust as needed. can't use raw sheet name in the output in case of special characters */
    proc import datafile="&fullFilePath." out=work.Imported_Sheet_&sheetNumber. dbms=&fileType. sheet="&sheet.";
    run;
    
  %end;
  
  /* TIP: You can do something like "format _char_ $256.;" in a data step, to set format for all character variables in a set. */
  /* Unfortunately length has to be set individually. Still helps a lot with merging these sheets together blindly. */
  
%mend importAllSheets;

/* call the macro. do not wrap the path in quotes, unless you add a "compress" at the start of the macro */
%importAllSheets(/root/folder/file.xlsx);
```
