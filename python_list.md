
### Sort two lists at the same time using the "decorate, sort, undecorate" idiom
```
list1, list2 = zip(*sorted(zip(list1, list2)))
```
