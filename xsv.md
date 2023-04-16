# xsv

[xsv](https://github.com/BurntSushi/xsv) is a really likable cli tool to analyze csvs. Examples are

```shell
# search column account_id for special values (regex)
# select from all columns the columns account_id and col_of_interest
# and render it in table format
# -d ';' sets the delimiter (only needed in first command)
xsv search -d ';' -s "account_id" "7202136853|7202288602|7201055060|7202136047|7202207637" large.csv | xsv select "account_id,col_of_interest" | xsv table | less -iS
```

```shell
# value count of columns:
# first restrict to col_of_interest (not necessary, but prevent unintersting
# other columns,
# then count every value with its frequency
xsv select -d ";" "col_of_interest" large.csv | xsv frequency
```
