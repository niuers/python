### Table of Contents
* [Basic Data Structures](#basic-data-structures)
* [Use `loc` and `iloc` to Select Data](#use-loc-and-iloc-to-select-data)
* [Avoid Using Apply Function](#avoid-using-apply-function)
* [Avoid Chained Indexing and prefer .loc/.iloc](#avoid-chained-indexing-and-prefer-lociloc)
* [Create new columns in a DataFrame]*(create-new-columns-in-a-dataframe)
* [Frequently used options when using pandas](#frequently-used-options-when-using-pandas)
* [Split Column of lists into multiple columns](#split-column-of-lists-into-multiple-columns)

## Basic Data Structures
### np.array/np.ndarray

### pd.Series

### pd.DataFrame


## Use `loc` and `iloc` to Select Data

* Example DataFrame
```
df = pd.DataFrame({'name':['a','b','c','d','e'], 'time':['Mon', 'Tue', 'Wed', 'Thur', 'Fri'],'value':[55,65,75,85,95]})
df = df.set_index('time')
print(df.head(10))

     name  value
time            
Mon     a     55
Tue     b     65
Wed     c     75
Thur    d     85
Fri     e     95
```
* Select data with `loc`
```
# pattern
# df.loc[rows, cols]
df.loc['Mon', 'value'] # return <class 'numpy.int64'>
df.loc[['Mon','Wed'], 'value'] # return <class 'pandas.core.series.Series'>
df.loc['Mon':'Wed', 'value']
df.loc[['Mon','Wed'], ['value', 'name']] # return <class 'pandas.core.frame.DataFrame'>
df.loc['Mon':'Wed', :]
df.loc[:, 'value']
df.loc['Mon', 'name':'value'] #return <class 'pandas.core.series.Series'>
df.loc['Mon':'Wed':2, :] #with step 2

# use slice
rows = slice('Mon', 'Wed', 2)
cols = ['name', 'value']
#These two are equivalent
df.loc[rows, cols]
df.loc['Mon':'Wed':2, cols]
```

* Select data with `iloc`
```
df.iloc[0,1]
df.iloc[1:3,:]
```


## [Avoid Using Apply Function](https://stackoverflow.com/questions/54432583/when-should-i-ever-want-to-use-pandas-apply-in-my-code)

#### What `apply` does?
* `DataFrame.apply` and `Series.apply` are convenience functions defined on `DataFrame` and `Series` object respectively.
* `pd.Series.apply` is a Python-level row-wise loop, ditto `pd.DataFrame.apply` row-wise (axis=1).
* There are very few situations where apply is appropriate to use . If you're not sure whether you should be using apply, you probably shouldn't.

#### Why is apply bad? 
* **It is because `apply` is slow**. `Pandas` makes no assumptions about the nature of your function, and so iteratively applies your function to each row/column as necessary. 
* Additionally, `apply` can handle so many situations means `apply` incurs some major overhead at each iteration. 
* Further, apply consumes a lot more memory, which is a challenge for memory bounded applications.


#### How to make code `apply` free?
* Numerical Data
  * Replace `apply` with **vectorized cython function**. e.g. use `df.sum()` instead of `df.apply(np.sum)`.
* String/Regex
  * Pandas provides "vectorized" string functions in most situations, but there are rare cases where those functions do not... "apply", so to speak.
  * The thing to note here is that iterative routines(e.g. list comprehension) happen to be faster than apply, because of the lower overhead. If you need to handle NaNs and invalid dtypes, you can build on this using a custom function you can then call with arguments inside the list comprehension.

#### A Common Pitfall: Exploding Columns of Lists
```
s = pd.Series([[1, 2]] * 3)
s.apply(pd.Series) # Horrible in performance
pd.DataFrame(s.tolist()) # Better
```

#### Are there any situations where apply is good?
* Apply is a convenience function, so there are situations where the overhead is negligible enough to forgive. It really depends on how many times the function is called. If you are using `pd.DataFrame.apply` row-wise, specifying raw=True (where possible) is often beneficial. 

* Functions that are Vectorized for Series, but not DataFrames
  * What if you want to apply a string operation on multiple columns? What if you want to convert multiple columns to datetime? These functions are vectorized for Series only, so they must be applied over each column that you want to convert/operate on.
  ```
  %timeit df.apply(pd.to_datetime, errors='coerce')   #admissible
  %timeit pd.to_datetime(df.stack(), errors='coerce').unstack()  #fine
  %timeit pd.concat([pd.to_datetime(df[c], errors='coerce') for c in df], axis=1) #fine
  %timeit for c in df.columns: df[c] = pd.to_datetime(df[c], errors='coerce') #fine

  ```
  * Note that it would also make sense to stack, or just use an explicit loop. All these options are slightly faster than using apply, but the difference is small enough to forgive.

#### Converting Series to str: astype versus apply
* Using apply to convert integers in a Series to string is comparable (and sometimes faster) than using astype.
* With floats, I see the astype is consistently as fast as, or slightly faster than apply. So this has to do with the fact that the data in the test is integer type.

#### GroupBy operations with chained transformations
* GroupBy.apply is also an iterative convenience function to handle anything that the existing GroupBy functions do not.
* But in general, `apply` is an acceptable solution if the goal is to reduce a `groupby` call (because `groupby` is also quite expensive).


#### Other Caveats

* It is also worth mentioning that apply operates on the first row (or column) twice. This is done to determine whether the function has any side effects. If not, apply may be able to use a fast-path for evaluating the result, else it falls back to a slow implementation.



## [Avoid Chained Indexing and prefer .loc/.iloc](https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html)
#### What is Chained Indexing?

Suppose we have following data
```
dfmi = pd.DataFrame([list('abcd'),
                     list('efgh'),
                     list('ijkl'),
                     list('mnop')],
                     columns=pd.MultiIndex.from_product([['one', 'two'],
                                                         ['first', 'second']]))

    one          two       
  first second first second
0     a      b     c      d
1     e      f     g      h
2     i      j     k      l
3     m      n     o      p
```

This is so called **'chained indexing'**
```
dfmi['one']['second']
```
This is the preferred use:
```
dfmi.loc[:, ('one', 'second')]
```
#### Issues with chain indexing

* Performance issue with Chained indexing

> dfmi['one'] selects the first level of the columns and returns a DataFrame that is singly-indexed. Then another Python operation dfmi_with_one['second'] selects the series indexed by 'second'. This is indicated by the variable dfmi_with_one because pandas sees these operations as separate events. e.g. separate calls to __getitem__, so it has to treat them as linear operations, they happen one after another.

> Contrast this to df.loc[:,('one','second')] which passes a nested tuple of (slice(None),('one','second')) to a single call to __getitem__. This allows pandas to deal with this as a single entity. Furthermore this order of operations can be significantly faster, and allows one to index both axes if so desired.

* Assignment may fail when using chained indexing

Assigning to the product of chained indexing has inherently unpredictable results. It may throw up the `SettingWithCopy` warning. To see this, think about how the Python interpreter executes this code:
```
dfmi.loc[:, ('one', 'second')] = value
# becomes
dfmi.loc.__setitem__((slice(None), ('one', 'second')), value)
```

but this code is handled differently:
```
dfmi['one']['second'] = value
# becomes
dfmi.__getitem__('one').__setitem__('second', value)
```

See that __getitem__ in there? Outside of simple cases, it’s very hard to predict whether it will return a view or a copy (it depends on the memory layout of the array, about which pandas makes no guarantees), and therefore whether the __setitem__ will modify dfmi or a temporary object that gets thrown out immediately afterward. That’s what SettingWithCopy is warning you about!

> dfmi.loc is guaranteed to be dfmi itself with modified indexing behavior, so dfmi.loc.__getitem__ / dfmi.loc.__setitem__ operate on dfmi directly. Of course, dfmi.loc.__getitem__(idx) may be a view or a copy of dfmi.

> Sometimes a SettingWithCopy warning will arise at times when there’s no obvious chained indexing going on. These are the bugs that SettingWithCopy is designed to catch! Pandas is probably trying to warn you that you’ve done this:

```
def do_something(df):
    foo = df[['bar', 'baz']]  # Is foo a view? A copy? Nobody knows!
    # ... many lines here ...
    # We don't know whether this will modify df or not!
    foo['quux'] = value
    return foo
```

* Evaluation order matters

When you use chained indexing, the order and type of the indexing operation partially determine whether the result is a slice into the original object, or a copy of the slice.

Pandas has the SettingWithCopyWarning because assigning to a copy of a slice is frequently not intentional, but a mistake caused by chained indexing returning a copy where a slice was expected.

* A chained assignment can also crop up in setting in a mixed dtype frame.

#### Correct and Incorrect ways using `loc` and `iloc` for assignment
  * case 1: 
  ```
  dfc.loc[0]['A'] = 1111 # won't work
  dfb[dfb.a.str.startswith('o')]['c'] = 42 # operates on a copy, won't work
  dfb['c'][dfb.a.str.startswith('o')] = 42 # works, but please avoid.
  ```
  * case 2: 
  ```
  dfc = dfc.copy() # sometimes worked, please avoid it!
  dfc['A'][0] = 111
  ```
  * case 3
  ```
  dfc.loc[0, 'A'] = 11 # Correct way to assign. N.B. This can't be used to create a new column called 'A'
  ```

## Create new columns in a DataFrame
* Directly assign

```
df[['new_col1', 'new_col2']] = pd.DataFrame({'A':[1,2,3], 'B':[4,5,6]})
```

## Frequently used options when using pandas
```python
pd.set_option('max_rows', 7)
pd.reset_option('max_rows')
```
### Display rows for a DataFrame
```
pd.set_option('display.max_rows', 1000)
display(df)
```

### Set display precision of numerical number with pandas
  * [temporaly set display precision](https://stackoverflow.com/questions/36909368/precision-lost-while-using-read-csv-in-pandas)
  ```python
  with pd.option_context('display.precision', 10):
      df.head()
  ```
  * set display precision for the whole session
  ```python
  pd.set_option('precision', 10)
  df.head()
  ```

## [Split Column of lists into multiple columns](https://stackoverflow.com/questions/35491274/pandas-split-column-of-lists-into-multiple-columns)

Suppose we have a DataFrame `df` with a column of lists:
```
       name
0	[ab, 1]
1	[cd, 2]
2	[ef, 3]
3	[gh, 4]
4	[ij, 5]
```
To split the column `name` into two columns, and assign to the original DataFrame

```
df[['char_name', 'name_num']] = pd.DataFrame(df['name'].values.to_list())
```
