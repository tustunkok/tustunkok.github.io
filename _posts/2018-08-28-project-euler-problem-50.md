---
layout: post
title:  "Project Euler - Problem 50"
categories: programming
---

# [Project Euler - Problem 50][1]
> The prime 41, can be written as the sum of six consecutive primes:
`41 = 2 + 3 + 5 + 7 + 11 + 13`
This is the longest sum of consecutive primes that adds to a prime below one-hundred.
The longest sum of consecutive primes below one-thousand that adds to a prime, contains 21 terms, and is equal to 953.
Which prime, below one-million, can be written as the sum of the most consecutive primes?

## Introduction
[Project Euler](https://projecteuler.net/) is a platform which presents challenging mathematical problems. By their definion:

> Project Euler exists to encourage, challenge, and develop the skills and enjoyment of anyone with an interest in the fascinating world of mathematics.

Those problems can be solved by hand or computer or any other thing that can calculate. In this notebook, I present you to a path to the solution of [Problem 50][1]. [Problem 50][1] asks for a prime number that can be written as the sum of the most consecutive prime numbers. To do this, we need:
1. all prime numbers that is below 1000000,
2. to create a window that contains consecutive prime numbers,
3. to determine the maximum range of consecutive prime numbers that also adds up to a prime number (and this number should below 1000000).

In doing so, I chose Python as my primary programming language and [Pandas](https://pandas.pydata.org/) library to help me working on data. This was necessary because I didn't want to waste my time while calculating the prime numbers. So, I downloaded all prime numbers less than 1000000 to a text file called *P-1000000.txt*. Anyway, let's get started. 

[1]: https://projecteuler.net/problem=50 "Problem 50"


```python
import pandas as pd
```

## Code
The first thing is to read all prime numbers to the memory. In csv file there is three columns. The first column is the rank of the prime number - which is basically the unique id of the number. The second column is the primary numbers' themselves and the third one is the difference between previous number and the current one. So, I give the column names to match with their properties since the file doesn't have one.


```python
primes = pd.read_csv('P-1000000.txt', header=None, names=['rank', 'prime', 'difference'])
primes.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>rank</th>
      <th>prime</th>
      <th>difference</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>5</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>7</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>11</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>




```python
max_range = 0
start = 0
stop = 0
num = 0
```

To gather the sums, we need to specify a window. This window consists of consecutive prime numbers. The lower end is specified by `i` and the upper end is specified by `j`. After getting the sum, the result is checked for to be a prime number. Finally, if all conditions are true, the range and the number's itself are saved. The last window that pass all these tests are the maximum range and our solution.


```python
for i in range(5):
    for j in range(600):
        # Get the sum of window.
        sum_of_consec_primes = primes['prime'][i:j].sum()
        if sum_of_consec_primes < 1000000:
            # If this sum is a prime number
            result = primes.loc[primes['prime'] == sum_of_consec_primes]
            if not result.empty and max_range < abs(j - i):
                max_range = abs(j - i)
                start = i
                stop = j
                num = sum_of_consec_primes
```


```python
print('Number is: {}. Start: {}, Stop: {}, Range: {}'.format(num, start, stop, stop - start))
```

    Number is: 997651. Start: 3, Stop: 546, Range: 543
    
