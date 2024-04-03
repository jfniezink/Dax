# Math & Statistical functions

SUM( column ) Adds all the numbers in a column  <br><br>
SUMX( table ,  expression ) Returns the sum of an expression evaluated for each row in a table<br><br>
AVERAGE( column ) Returns the average (arithmetic meanG of all the numbers in a column<br><br>
AVERAGEX( table ,  expression ) Calculates the average (arithmetic mean) of a set of expressions evaluated over a table<br><br>
MEDIAN( column ) Returns the median of a column<br><br>
MEDIANX( table ,  expression ) Calculates the median of a set of expressions evaluated over a table<br><br>
GEOMEAN( column ) Calculates the geometric mean of a column<br><br>
GEOMEANX( table ,  expression ) Calculates the geometric mean of a set of expressions evaluated over a table<br><br>
COUNT( column ) Returns the number of cells in a column that contain non-blank values<br><br>
COUNTX( table ,  expression ) Counts the number of rows from an expression that evaluates to a non-blank value<br><br>
DIVIDE( numerator ,  denominator  [, alternateresult ]) Performs division and returns alternate result or BLANK() on division by 0<br><br>
MIN( column ) Returns a minimum value of a column<br><br>
MAX( column ) Returns a maximum value of a column<br><br>
COUNTROWS([ table ]) Counts the number of rows in a table<br><br>
DISTINCTCOUNT( column ) Counts the number of distinct values in a column<br><br>
RANKX( table ,  expression [,  value [,  order [,  ties ]]]) Returns the ranking of a number in a list of numbers for each row in the table argument. <br>
# Filter functions
FILTER( table ,  filter ) Returns a table that is a subset of another table or expression<br><br>
CALCULATE( expression [,  filter1  [,  filter2  [, …]]]) Evaluates an expression in a filter context<br><br>
HASONEVALUE( columnName ) Returns TRUE when the context for columnName has been filtered down to one distinct value only. Otherwise it is FALSE<br><br>
ALLNOBLANKROW( table  |  column [,  column [,  column [,…]]]) Returns a table that is a subset of another table or expression<br><br>
ALL([ table  |  column [,  column [,  column [,…]]]]) Returns all the rows in a table, or all the values in a column, ignoring any filters that might have been applied<br><br>
ALLEXCEPT( table ,  column [,  column [,.]]) Returns all the rows in a table except for those rows that are affected by the specified column filters<br><br>
REMOVEFILTERS([ table  |  column ][,  column [,  column [,…]]]]) Clear all filters from designated tables or columns. <br><br>

# Logical functions 

IF( logical_test ,  value_if_true [,  value_if_false ]) Checks a condition, and returns a certain value depending on whether it is true or false<br><br>
AND( logical 1 ,  logical 2 ) Checks whether both arguments are TRUE, and returns TRUE if both arguments are TRUE. Otherwise, it returns FALSE<br><br>
OR( logical 1 ,  logical 2 ) Checks whether one of the arguments is TRUE to return TRUE. The function returns FALSE if both arguments are FALSE<br><br>
NOT( logical ) Changes TRUE to FALSE and vice versa<br><br>
SWITCH( expression ,  value ,  result [,  value ,  result ]…[,  else ]) Evaluates an expression against a list of values and returns one of possible results<br><br>
IFERROR( value ,  value_if_error ) Returns value_if_error if the first expression is an error and the value of the expression itself otherwise. <br><br>

# Date & time functions

CALENDAR( start_date ,  end_date ) Returns a table with a single column named "%ate" that contains a contiguous set of dates<br><br>
DATE( year ,  month ,  day ) Returns the specified date in datetime format<br><br>
DATEDIFF( date_1 ,  date_2 ,  interval ) Returns the number of units between two dates as defined in  interval <br><br>
DATEVALUE( date_text ) Converts a date in text to a date in datetime format<br><br>
DAY( date ) Returns a number from 1 to 31 representing the day of the month<br><br>
WEEKNUM( date ) Returns weeknumber in the year<br><br>
MONTH( date ) Returns a number from 1 to 12 representing a month<br><br>
QUARTER( date ) Returns a number from 1 to 4 representing a quarter. <br><br>

# Time intelligence functions

DATEADD( dates ,  number_of_intervals ,  interval ) Moves a date by a specific interval<br><br>
DATESBETWEEN( dates ,  date_1 ,  date_2 ) Returns the dates between specified dates<br><br>
TOTALYTD( expression ,  dates [,  filter ][,  year_end_date ]) Evaluates the year-to-date value of the expression in the current context<br><br>
SAMEPERIODLASTYEAR( dates ) Returns a table that contains a column of dates shifted one year back in time<br><br>
STARTOFMONTH( dates ) // ENDOFMONTH( dates ) Returns the start // end of the month<br><br>
STARTOFQUARTER( dates ) // ENDOFQUARTER( dates ) Returns the start // end of the quarter<br><br>
STARTOFYEAR( dates ) // ENDOFYEAR( dates ) Returns the start // end of the quarter. <br><br>

# Relationship functions

CROSSFILTER( left_column ,  right_column ,  crossfiltertype ) Specifies the cross-filtering direction to be used in a calculation<br><br>
RELATED( column ) Returns a related value from another table. <br><br>

# Table manipulation functions

SUMMARIZE( table ,  groupBy_columnName [,  groupBy_columnName ]…[,  name ,  expression ]…)  Returns a summary table for the requested totals over a set of groups<br><br>
DISTINCT( table ) Returns a table by removing duplicate rows from another table or expression<br><br>
ADDCOLUMNS( table ,  name ,  expression [,  name ,  expression ]…) Adds calculated columns to the given table or table expression<br><br>
SELECTCOLUMNS( table ,  name ,  expression [,  name ,  expression ]…) Selects calculated columns from the given table or table expression<br><br>
GROUPBY( table  [,  groupBy_columnName [, [ column_name ] [ expression ]]…) Create a summary of the input table grouped by specific columns<br><br>
INTERSECT( left_table ,  right_table ) Returns the rows of the left-side table that appear in the right-side table<br><br>
NATURALINNERJOIN( left_table ,  right_table ) Joins two tables using an inner join<br><br>
NATURALLEFTOUTERJOIN( left_table ,  right_table ) Joins two tables using a left outer join<br><br>
UNION( table ,  table [,  table  [,…]]) Returns the union of tables with matching columns. <br><br>

# Text functions

EXACT( text_1 ,  text_2 ) Checks if two strings are identical (EXACT() is case sensitive) <br><br>
FIND( text_tofind ,  in_text ) Returns the starting position a text within another text (FIND() is case sensitive) <br><br>
FORMAT( value ,  format ) Converts a value to a text in the specified number format<br><br>
LEFT( text ,  num_chars ) Returns the number of characters from the start of a string. <br><br>
RIGHT( text ,  num_chars ) Returns the number of characters from the end of a string<br><br>
LEN( text ) Returns the number of characters in a string of text<br><br>
LOWER( text ) Converts all letters in a string to lowercase<br><br>
UPPER( text ) Converts all letters in a string to uppercase<br><br>
TRIM( text ) Remove all spaces from a text string<br><br>
CONCATENATE( text_1 ,  text_2 ) Joins two strings together into one string<br><br>
SUBSTITUTE( text ,  old_text ,  new_text ,  instance_num ) Replaces existing text with new text in a string<br><br>
REPLACE( old_text ,  start_posotion ,  num_chars ,  new_text ) Replaces part of a string with a new string<br>

# Information functions

COLUMNSTATISTICS() Returns statistics regarding every column in every table. This function has no arguments <br><br>
NAMEOF( value ) Returns the column or measure name of a value<br><br>
ISBLANK( value ) // ISERROR( value ) Returns whether the value is blank // an error<br><br>
ISLOGICAL( value ) Checks whether a value is logical or not<br><br>
ISNUMBER( value ) Checks whether a value is a number or not<br><br>
ISFILTERED( table  |  column ) Returns true when there are direct filters on a column<br><br>
ISCROSSFILTERED( table  |  column ) Returns true when there are crossfilters on a column<br><br>
USERPRINCIPALNAME() Returns the user principal name or email address. This function has no arguments. <br><br>

# Dax statements 

VAR( name  =  expression ) Stores the result of an expression as a named variable. To return the variable, use RETURN after the variable is defined<br><br>
COLUMN( table [ column ] =  expression ) Stores the result of an expression as a column in a table. <br><br>
ORDER BY( table [ column ]) %efines the sort order of a column. Every column can be sorted in ascending (ASCG or descending (DESC) way. <br><br>

# Dax operators
| Comparison operators  | Meaning  |
|:-:|:-:|
| = | Equal to |
| == | Strict equal to |
| > | Greater than |
| < | Smaller than |
| >= | Greater than or equal to |
| =< | Smaller than or equal to |
| <> | Not equal to |

| Text operator | Meaning | Example |
|:-:|:-:|:-:|
| & | Concatenates tekst values | Concatenates text values | [City]&", "&[State] |

| Logical operator | Meaning | Example |
|:-:|:-:|:-:|
| && | AND conditon | ([City] = "Bru") && ([Return] = "Yes"))  |
| || | OR condition | ([City] = "Bru") || ([Return] = "Yes"))  |
| IN {} | OR condition for each row | Product[Color] IN {"Red", "Blue", "Gold"} |
