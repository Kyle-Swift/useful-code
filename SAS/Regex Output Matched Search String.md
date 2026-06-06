I couldn't find an answer to this question online, but I managed to piece the solution together... pretty proud of myself.

## My use case:
We have a prxmatch function which uses regex to search for many different substrings at once. We want to see which one was matched on, for analytical purposes.

Prxchange allows us to substitute the matched part of the string with something else.

By default, prxchange ignores anything that wasn't part of the match, and only replaces the bit that was found.

## Solution:
```SAS
data matched;
set unfiltered_list;

    if _n_ = 1 then
      do;
        retain contains_regex
               replace_regex;

        contains_regex = PRXPARSE("/(TRAVEL|FINAN|ACE)/");
        replace_regex = PRXPARSE("s/(.*|^)(\sTRAVEL|FINAN|ACE)(.*|$)/$2/");

      end;

if prxmatch(contains_regex,SearchTerm)>0 then do;
  MatchingTerm = prxchange(replace_regex,1,SearchTerm);
  output;
end;

run;
```

## To break down the substitution string:
```
s/(.*|^)(\sTRAVEL|FINAN|ACE)(.*|$)/$2/

s///                  Used to replace a matched term with something
(.*|^)                Any amount of any characters, or the beginning of the string
(\sTRAVEL|FINAN|ACE)  My list of search terms
(.*|$)                Any amount of any characters, or the end of the string
/$2/                  The replacement text is going to be the matched term from the second set of parentheses
```

Having the outer (.*) matches, and then replacing with $2 is necessary.

If you omit those, and only search for your term list. Then, for ease of explaining, replace '$2' with 'FOO':
- The prxchange function will only replace the search term that was found with 'FOO'.
- The parts of the string <b>outside</b> the matched term, are not included in the prxchange process. They remain untouched no matter what the replacement string is.

Including the (.*|^) and (.*|$) pieces makes it so that the entire input string is part of the matched text.

## Important note:
The '1' in 'prxchange(replace_regex,1,SearchTerm)' means it will stop after finding the first complete match.

This is desireable. Many use cases use '-1' to replace ALL matched terms with something, but we don't want that, in this case.
