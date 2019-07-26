### [Avoid Chained Indexing and prefer .loc](https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html)

Suppose we have following data
```
In [338]: dfmi = pd.DataFrame([list('abcd'),
   .....:                      list('efgh'),
   .....:                      list('ijkl'),
   .....:                      list('mnop')],
   .....:                     columns=pd.MultiIndex.from_product([['one', 'two'],
   .....:                                                         ['first', 'second']]))
   .....: 

In [339]: dfmi
Out[339]: 
    one          two       
  first second first second
0     a      b     c      d
1     e      f     g      h
2     i      j     k      l
3     m      n     o      p
```

This is so called 'chained indexing'
```
dfmi['one']['second']
```
This is the preferred use:
```
dfmi.loc[:, ('one', 'second')]
```
* Performance issue with Chained indexing

> dfmi['one'] selects the first level of the columns and returns a DataFrame that is singly-indexed. Then another Python operation dfmi_with_one['second'] selects the series indexed by 'second'. This is indicated by the variable dfmi_with_one because pandas sees these operations as separate events. e.g. separate calls to __getitem__, so it has to treat them as linear operations, they happen one after another.

> Contrast this to df.loc[:,('one','second')] which passes a nested tuple of (slice(None),('one','second')) to a single call to __getitem__. This allows pandas to deal with this as a single entity. Furthermore this order of operations can be significantly faster, and allows one to index both axes if so desired.

* Assignment fail when using chained indexing

Assigning to the product of chained indexing has inherently unpredictable results. It may throw up the SettingWithCopy warning. To see this, think about how the Python interpreter executes this code:
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

* Correct way to assign using `loc` and `iloc`
  * case 1: 
  ```
  dfc.loc[0]['A'] = 1111 # won't work
  dfb[dfb.a.str.startswith('o')]['c'] = 42 # operates on a copy, won't work
  dfb['c'][dfb.a.str.startswith('o')] = 42 # works, but please avoid.
  ```
  * case 2: 
```
In [347]: dfc = dfc.copy() # sometimes worked, please avoid it!
In [348]: dfc['A'][0] = 111
```

  * case 3
  ```
  dfc.loc[0, 'A'] = 11 #Correct way to assign
  ```

### Frequently used options when using pandas
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
