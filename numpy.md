# Table of Contents

* [](#)
* [](#)

1. What is numpy broadcasting?
1. Why numpy vectorization is very efficient?

# Numpy Broadcasting

* The term broadcasting describes how numpy treats arrays with different shapes during arithmetic operations.
* Subject to certain constraints, the smaller array is “broadcast” across the larger array so that they have compatible shapes. 
* Broadcasting provides a means of vectorizing array operations so that looping occurs in C instead of Python.
* It does this without making needless copies of data and usually leads to efficient algorithm implementations. There are also cases where broadcasting is a bad idea because it leads to inefficient use of memory that slows computation.

## The Broadcasting Rule
* In order to broadcast, the size of the trailing axes for both arrays in an operation must either be the same size or one of them must be one.

