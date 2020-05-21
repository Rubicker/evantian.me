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
