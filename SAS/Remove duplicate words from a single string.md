"Word" means the same as "Substring" in this case. The example code is actually looking for distinct words as commas delimmited substrings.

Process flows as:
  1. New string = First word in original string.
  2. Do (from the second word until the last):
      1. Pick the next word.
      2. If this word isn't already in the new string, keep it. Otherwise, move on.

```SAS
data want;
  set have;
  
  newString=scan(oldString, 1, ','); /* first word */
  
  do i=2 to countw(oldString, ','); /* iterate for every other word */
    word=scan(oldString, i, ','); /* isolate the word */
    found=find(newString, word, 'it'); /* check if the current word has already been captured */
    if found=0 then newString=catx(',', newString, word); /* if this is the first occurence of this word, keep it */
  end;
  
  drop word oldString found i;
  
run;
```
