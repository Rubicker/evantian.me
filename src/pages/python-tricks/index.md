---

title: Advanced Python Skills

date: '2020-05-21'

spoiler: Things I learnt from Python one-liners.

---

Here I'll record some tricks learnt from a book named [Python one-liner](https://www.amazon.com/Python-One-Liners-Christian-Mayer-ebook/dp/B07ZY7XMX8):

## List Comprehension

### Using List Comprehension to Find Top Earners

> Pick up all staff members who earn at least $100,000 per year. Your desired output is a list
> of tuples, each consisting of two values: the employee name and the employee's yearly salary.

```python
# This is the original data
employees = {
  'Alice': 100000,
  'Bob': 99817,
  'Carol': 122908,
  'Frank': 88123,
  'Eve': 93121
}
```

Python offers a powerful way of creating new lists: *list comprehension*

`[expression + context]`

We could do these:

```python
# output: [(0, 0), (0, 1), [1, 0], [1, 1]]
[(x, y) for x in range(2) for y in range(2)]

# output: [1, 9, 25, 49, 81]
[x ** 2 for x in range(10) if x % 2 > 0]

# output: ['i', 'am', 'not', 'robot']
[x.lower() for x in ['I', 'AM', 'NOT', 'ROBOT']]
```

So the answer will be:

```python
[(name, salary) for name, salary in employees.item() if salary >= 100000]
```

### Using List Comprehension to Find Words with High Information Value

> Given a multiline string, create a list of lists - each consisting of all the words in a line that have more than three characters

```python
# Data
text = '''
Call me Ishmael. Some years ago - never mind how long precisely - having
little or no money in my purse, and nothing particular to interest me
on shore, I thought I would sail about a little and see the watery part
of the world. It is a way I have of driving off the spleen, and regulating
the circulation. - Moby Dick'''
```

```python
w = [
  [word for word in line.split() if len(word) > 3] 
  for line in text.split('\n')
]
```

## Reading Files

It's a good practice to close the file after reading it with the command `f.close()`, to ensure all the data is properly written into the file instead of residing in temporary memory. However, in a few exceptions, Python closed the fil automatically: one of these exceptions occurs when the reference count drops to zero, as you'll see in the following code:

```python
# readFile.py
[line.strip() for line in open("readFile.py")]
```


## Lamda and Map Functions

Lamda functions allow you to define a new function in a single line by using the keyword `lamda`. This is useful when you want to quickly create a function that you'll use once and can be garbege-collected immediately afterward:

`lamda arguments: return expression`

```python
lamda x, y: x + y
```

> Given a list of strings, return a new list of tuples, each consisting of a Boolean value and the original string. The Boolean value indicates whether the string 'anonymous' appears in the original string


```python
# Data
txt = [
  'lambda functions are anonymous functions.',
  'anonymous functions dont have a name.',
  'functions are objects in Python.'
]

mark = map(lamda s: (True, s) if 'anonymous' in s else (False, s), txt)

# or use list comprehension
mark = [(True, s) for s in txt if 'anonymous' in s]
```

## Slicing

Basic syntax:

`x[start:stop:step]`

If you don't include the `step` argument, Python assumes the default step size of one; <br />
if you don't include the `start` or `stop` argument, Python assumes you want start at the start, or end at the end; <br />
If you set `step` as a negative number, the slice traverses the sequence in reverse order.

```python
# Data
letters_amazon = '''
We spent several years building our own database engine,
Amazon Aurora, a fully-managed MySQL and PostgreSQL-compatible
service with the same or better durability and availability as
the commercial engines, but at one-tenth of the cost. We were
not surprised when this worked. '''

find = lamda x, q: x[x.find(q) - 18: x.find(q) + 18] if q in x else -1
```

## Combining List Comprehension and Slicing

1. To create a new training data sample from our data-a list of lists, each consisting of six floats-by including only every other float value from the original data set:

```python
# data
price = [[9.9, 9.8, 9.8, 9.4, 9.5, 9.7],
         [9.5, 9.4, 9.4, 9.3, 9.2, 9.1],
         [8.4, 7.9, 7.9, 8.1, 8.0, 8.0],
         [7.1, 5.9, 4.8, 4.8, 4.7, 3.9]]

sample = [line[::2] for line in price] 
```

2. Our data is a dictionary of dictionaries storing the hourly wages of company employees. You want to extract a list of the companies paying below your state's minimum wage(< $9) for at least one employee:

```python
# data
companies = {
  'CoolCompany' : {'Alice' : 33, 'Bob' : 28, 'Frank' : 29},
  'CheapCompany' : {'Ann' : 4, 'Lee' : 9, 'Chrisi' : 7},
  'SosoCompany' : {'Esther' : 38, 'Cole' : 8, 'Paris' : 18}
}

illegal = [x for x in companies if any(y < 9 for y in companies[x].values())]
```

## `zip()`

The `zip()` function takes iterables *iter_1*, *iter_2*, ..., *iter_n* and aggregates them into a single iterable by aligning the corresponding i-th values into a single tuple.