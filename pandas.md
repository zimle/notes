# Pandas

## Datetime

```python
# convert date column as string to a datetime64[ns] column
df.MY_DATE_COLUMN = pd.to_datetime(df.MY_DATE_COLUMN, format="ISO8601")
# truncate datetime64[ns] to month, count occurances and sort by date; output as markdown
(df.MY_DATE_COLUMN.dt.to_period('M')).value_counts().sort_index().to_markdown()
```

## Merging dataframes

```python
merged = df.merge(mods, left_on=["ACCOUNT_KEY"], right_on=["account"])
```

## Print out whole dataframe

Sometimes, one wants to see the whole output of a dataframe or series. The most elegant way to do this is to use a context manager:

```python
with pd.option_context('display.max_rows', None,
                       'display.max_columns', None,
                       'display.precision', 3,
                       ):
    print(df)
```

This way, the global environment is not polluted by some single large chunk you want to see.
