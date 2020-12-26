

show functions;





| **Return Type**   | **Function Name (Signature)**                    | **Description**                                              |
| :---------------- | :----------------------------------------------- | :----------------------------------------------------------- |
| BIGINT            | round(double a)                                  | returns the rounded BIGINT value of the double               |
| BIGINT            | floor(double a)                                  | returns the maximum BIGINT value that is equal or less than the double |
| BIGINT            | ceil(double a)                                   | returns the minimum BIGINT value that is equal or greater than the double |
| double            | rand(), rand(int seed)                           | returns a random number (that changes from row to row). Specifiying the seed will make sure the generated random number sequence is deterministic. |
| string            | concat(string A, string B,...)                   | returns the string resulting from concatenating B after A. For example, concat('foo', 'bar') results in 'foobar'. This function accepts arbitrary number of arguments and return the concatenation of all of them. |
| string            | substr(string A, int start)                      | returns the substring of A starting from start position till the end of string A. For example, substr('foobar', 4) results in 'bar' |
| string            | substr(string A, int start, int length)          | returns the substring of A starting from start position with the given length, for example, substr('foobar', 4, 2) results in 'ba' |
| string            | upper(string A)                                  | returns the string resulting from converting all characters of A to upper case, for example, upper('fOoBaR') results in 'FOOBAR' |
| string            | ucase(string A)                                  | Same as upper                                                |
| string            | lower(string A)                                  | returns the string resulting from converting all characters of B to lower case, for example, lower('fOoBaR') results in 'foobar' |
| string            | lcase(string A)                                  | Same as lower                                                |
| string            | trim(string A)                                   | returns the string resulting from trimming spaces from both ends of A, for example, trim(' foobar ') results in 'foobar' |
| string            | ltrim(string A)                                  | returns the string resulting from trimming spaces from the beginning(left hand side) of A. For example, ltrim(' foobar ') results in 'foobar ' |
| string            | rtrim(string A)                                  | returns the string resulting from trimming spaces from the end(right hand side) of A. For example, rtrim(' foobar ') results in ' foobar' |
| string            | regexp_replace(string A, string B, string C)     | returns the string resulting from replacing all substrings in B that match the Java regular expression syntax(See [Java regular expressions syntax](http://java.sun.com/j2se/1.4.2/docs/api/java/util/regex/Pattern.html)) with C. For example, regexp_replace('foobar', 'oo\|ar', ) returns 'fb' |
| int               | size(Map<K.V>)                                   | returns the number of elements in the map type               |
| int               | size(Array<T>)                                   | returns the number of elements in the array type             |
| *value of <type>* | cast(*<expr>* as *<type>*)                       | converts the results of the expression expr to <type>, for example, cast('1' as BIGINT) will convert the string '1' to it integral representation. A null is returned if the conversion does not succeed. |
| string            | from_unixtime(int unixtime)                      | convert the number of seconds from the UNIX epoch (1970-01-01 00:00:00 UTC) to a string representing the timestamp of that moment in the current system time zone in the format of "1970-01-01 00:00:00" |
| string            | to_date(string timestamp)                        | Return the date part of a timestamp string: to_date("1970-01-01 00:00:00") = "1970-01-01" |
| int               | year(string date)                                | Return the year part of a date or a timestamp string: year("1970-01-01 00:00:00") = 1970, year("1970-01-01") = 1970 |
| int               | month(string date)                               | Return the month part of a date or a timestamp string: month("1970-11-01 00:00:00") = 11, month("1970-11-01") = 11 |
| int               | day(string date)                                 | Return the day part of a date or a timestamp string: day("1970-11-01 00:00:00") = 1, day("1970-11-01") = 1 |
| string            | get_json_object(string json_string, string path) | Extract json object from a json string based on json path specified, and return json string of the extracted json object. It will return null if the input json string is invalid. |



常见的聚合函数

| **Return Type** | **Aggregation Function Name (Signature)**             | **Description**                                              |
| :-------------- | :---------------------------------------------------- | :----------------------------------------------------------- |
| BIGINT          | count(*), count(expr), count(DISTINCT expr[, expr_.]) | count(*)—Returns the total number of retrieved rows, including rows containing NULL values; count(expr)—Returns the number of rows for which the supplied expression is non-NULL; count(DISTINCT expr[, expr])—Returns the number of rows for which the supplied expression(s) are unique and non-NULL. |
| DOUBLE          | sum(col), sum(DISTINCT col)                           | returns the sum of the elements in the group or the sum of the distinct values of the column in the group |
| DOUBLE          | avg(col), avg(DISTINCT col)                           | returns the average of the elements in the group or the average of the distinct values of the column in the group |
| DOUBLE          | min(col)                                              | returns the minimum value of the column in the group         |
| DOUBLE          | max(col)                                              | returns the maximum value of the column in the group         |





- Positive
- Negative
- Addition
- Subtraction
- Multiplication
- Division
- Average (avg)
- Sum
- Count
- Modulus (pmod)
- Sign – [Hive 0.13.0](https://issues.apache.org/jira/browse/HIVE-6246) and later
- Exp – [Hive 0.13.0](https://issues.apache.org/jira/browse/HIVE-6327) and later
- Ln – [Hive 0.13.0](https://issues.apache.org/jira/browse/HIVE-6327) and later
- Log2 – [Hive 0.13.0](https://issues.apache.org/jira/browse/HIVE-6327) and later
- Log10 – [Hive 0.13.0](https://issues.apache.org/jira/browse/HIVE-6327) and later
- Log(*base*) – [Hive 0.13.0](https://issues.apache.org/jira/browse/HIVE-6327) and later
- Sqrt – [Hive 0.13.0](https://issues.apache.org/jira/browse/HIVE-6327) and later
- Sin – [Hive 0.13.0](https://issues.apache.org/jira/browse/HIVE-6327) and later
- Asin – [Hive 0.13.0](https://issues.apache.org/jira/browse/HIVE-6327) and later
- Cos – [Hive 0.13.0](https://issues.apache.org/jira/browse/HIVE-6327) and later
- Acos – [Hive 0.13.0](https://issues.apache.org/jira/browse/HIVE-6327) and later
- Tan – [Hive 0.13.0](https://issues.apache.org/jira/browse/HIVE-6327) and later
- Atan – [Hive 0.13.0](https://issues.apache.org/jira/browse/HIVE-6327) and later
- Radians – [Hive 0.13.0](https://issues.apache.org/jira/browse/HIVE-6327) and later
- Degrees – [Hive 0.13.0](https://issues.apache.org/jira/browse/HIVE-6385) and later

These rounding functions can also take decimal types:

- Floor
- Ceiling
- Round



