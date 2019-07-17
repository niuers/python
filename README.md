# python
Notes for Python Programming


* Set display precision of numerical number with pandas
  * [temporaly set display precision](https://stackoverflow.com/questions/36909368/precision-lost-while-using-read-csv-in-pandas)
  ```python
  with pd.option_context('display.precision', 10):
      df.head()
  ```
