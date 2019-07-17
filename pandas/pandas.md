* Frequently used options when using pandas
```python
pd.set_option('max_rows', 7)
pd.reset_option('max_rows')

```
* Set display precision of numerical number with pandas
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
