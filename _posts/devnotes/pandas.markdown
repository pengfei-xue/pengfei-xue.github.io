## iter rows over dataframe

    >>> df = pd.DataFrame([[1, 1.5]], columns=['int', 'float'])
    >>> row = next(df.iterrows())[1]
    >>> row
    int      1.0
    float    1.5
    Name: 0, dtype: float64
    >>> print(row['int'].dtype)
    float64
    >>> print(df['int'].dtype)
    int64

[pandas.DataFrame.iterrows](http://pandas.pydata.org/pandas-docs/version/0.17.0/generated/pandas.DataFrame.iterrows.html)
