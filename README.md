Testing (with Python)
=====================
* importance and usefulness are well known
* but still often unsued
* two major problems (beside many more)
 * inconvenient to write and need for boilerplate code
 * seen as useful but optional tool
* try to tackle both problems
 * py.test &rarr; easy to write and use tests for python
 * testing as a essential part of software development


##### testing in this context = automatic tests

what?
=====
* everything 
 * from single functions to executing the whole application
* unit test - smallest unit of functionality (function/method)
* integration test - combination of units
* functional test - particular feature
* system test - integrated system



why testing?
============
* verify (written) code
  * expected behaviour
* also usefull in reverse to demonstrate errors, bugs and missing functionality
 * e.g. in bug fixing
 * write test that exploits the bug
 * fix the bug
 * test again to verify the fix
* confidence-building benefits
 * Did I break it?
 * Can I fix it?
 * Lets try it!
* obtain new view on your code
 * modularity and coupling
 * API and overal design
 * code smells
 * **good** tests &rarr; maintainable and reusable code (&rarr; better code?!)
* tests as additional specs and documentation
 * reimplement software by using the tests



how to test?
============
`mymodule.py`

```python
def mean(list_of_numbers):
    """Return mean of a list of numbers."""
    return sum(list_of_numbers) / len(list_of_numbers)
```

### 1. fire up the python shell

```python
>>> from mymodule import mean
>>> mean([10, 20])
15
>>> mean([100, 300])
200
```

### 2. make it a file

```python
if __name__ == '__main__':
    print mean([10, 20]), 'should be 15'
    print mean([100, 300]), 'should be 200'
```

```
$ python mymodule.py
15 should be 15
200 should be 200
```

### 3. the `assert` statement
```python
>>> assert True
>>> assert 2 > 1
>>> assert True != False
```

```python
>>> assert 1 == 2
AssertionError
```

```python
if __name__ == '__main__':
    assert mean([10, 20]) == 15
    assert mean([100, 300]) == 200
```

### 4. use `unittest`
```python
import unittest
from mymodule import mean

class TestMymodule(unittest.TestCase):

    def test_mean(self):
        self.assertEqual(mean([10, 20]), 15)
        self.assertEqual(mean([100, 300]), 200)

 
if __name__ == '__main__':
    unittest.main()
```

```
$ python unit.py
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
```

#### failing test

```python
self.assertEqual(mean([1, 2]), 1.5)
```

```
$ python unit.py
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
```

```
$ python unit.py
F
======================================================================
FAIL: test_mean (__main__.TestMymodule)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "unit.py", line 9, in test_mean
    self.assertEqual(mean([1, 2]), 1.5)
AssertionError: 1 != 1.5

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
```


function | statement
---------|----------
assertEqual(a, b) | 	a == b 	 
assertNotEqual(a, b) | 	a != b 	 
assertTrue(x) |	bool(x) is True 	 
assertFalse(x) |	bool(x) is False 	 
assertIs(a, b) |	a is b 	*
assertIsNot(a, b)| 	a is not b 	*
assertIsNone(x) |	x is None 	*
assertIsNotNone(x) |	x is not None 	*
assertIn(a, b) |	a in b 	*
assertNotIn(a, b) |	a not in b 	*
assertIsInstance(a, b)| 	isinstance(a, b) 	*
assertNotIsInstance(a, b)| 	not isinstance(a, b) 	*

\* Python 2.7+


py.test
=======

* Holger Krekel (pypy)
* combines testrunner with the assert statement

`test_mymodule.py`

```python
from mymodule import mean


def test_mean():
    assert mean([10, 20]) == 15
    assert mean([100, 300]) == 200
```

```
$ py.test
============================= test session starts ==============================
platform linux2 -- Python 2.6.6 -- pytest-2.3.5
collected 1 items 

test_mymodule.py .

=========================== 1 passed in 0.02 seconds ==========================
```

#### failing test


```
$ py.test 
============================= test session starts ==============================
platform linux2 -- Python 2.6.6 -- pytest-2.3.5
collected 1 items 

test_mymodule.py F

=================================== FAILURES ===================================
__________________________________ test_mean ___________________________________

    def test_mean():
        assert mean([10, 20]) == 15
        assert mean([100, 300]) == 200
>       assert mean([1, 2]) == 1.5
E       assert 1 == 1.5
E        +  where 1 = mean([1, 2])

test_mymodule.py:7: AssertionError
=========================== 1 failed in 0.01 seconds ===========================
```

**py.test runs unittest, nose and even doctest**

`mymodule.py`
```python
def mean(list_of_numbers):
    """
    Return mean of a list of numbers.
    
    >>> mean([20, 40])
    30
    """
    return sum(list_of_numbers) / len(list_of_numbers)
```

```
$ py.test -v --doctest-modules
============================= test session starts ==============================
platform linux2 -- Python 2.6.6 -- pytest-2.3.5 -- /usr/bin/python
collected 3 items 

mymodule.py: [doctest] PASSED
test_mymodule.py: [doctest] PASSED
test_mymodule.py:4: test_mean PASSED

=========================== 1 passed in 0.02 seconds ===========================
```

* `py.test -v` verbose output
* `py.test -s` print stdout/stderr
* `py.test -k <pattern>` only execute tests that match <pattern>
* **fixtures** (e.g. get fake files, monkeypatch stdlib, mock server, ...)

#### expect Exceptions

```python
import pytest
from mymodule import mean

def test_mean_zero_divison():
    with pytest.raises(ZeroDivisionError):
        mean([])
```

when to test?
=============

`Always and everything.`

* if you don't test it, it does not exist
 * test that are not executed == no tests at all
* Test Driven Development (TDD)
 * tests before code
* legacy code
 * test when you create something new
 * test when you modify existing code

testable code
=============

### problems when
* implict dependencies &rarr; dependency injection
 * reaching out to other classes

```python
def get_everything():
    db = database.get()
    
    return db.select(*)
```
&darr;
```python
def proc_everything(db):
    return db.select(*)
```


* functions that are easy to test
 * have **no side effects**
 * no implicit dependencies
 * only try to solve one thing (Single responsibility principle) vs. **the god function**

**Functions are fun. Write lots of the them, but keep them small and clear in purpose.**

* Does your class really need all those methods?
 * Most of the time you can turn them into pure functions with just one extra parameter

* Dependency Injection instead of implicit dependencies




red herings
===========

* big functions and classes - too much is going on &rarr; split into separate ones
* many methods - are they really state dependend? &rarr; turn them into functions

conclusion
==========

* write many, small and clear functions 
* test as often as possible
* test as much as possible 
* write tests for what you add or change
* write tests before you code
* don't rely blindly on your tests

coverage
========

py.test und coverage


* install pytest-cov `pip install pytest-cov`
* py.test finds it

```
py.test --cov . test_all.py --cov-report=html --cov-report=term
```

* not sure about how . and test_all are interacting
* but with this it test every py module in place
* but it seems like its not running the authors.py

```
coverage run --branch `which py.test`
```

diff-coverage
* https://github.com/edx/diff-cover

mocking
=======

* http://www.voidspace.org.uk/python/mock/getting-started.html
* `pip install mock`

### where to patch
* http://www.voidspace.org.uk/python/mock/patch.html#where-to-patch
* scenario

```
a.py
    -> Defines SomeClass

b.py
    -> from a import SomeClass
    -> use SomeClass
```

do `@patch(‘b.SomeClass’)`

but if its

```
b.py
    -> import a impo
    -> use a.SomeClass
```

do `@patch(‘a.SomeClass’)`

talks
====
* https://www.youtube.com/watch?v=ukm64IUANwE
