# Error classes

pytype has the following classes of errors, which can be disabled with a
`pytype: disable=error-class` directive. For example, to suppress an
error for a missing attribute `foo`:

```
x = a.foo  # pytype: disable=attribute-error
```

or, to suppress all attribute errors for a block of code:

```
# pytype: disable=attribute-error
x = a.foo
y = a.bar
# pytype: enable=attribute-error
```

See [Silencing Errors][silencing-errors] for a more detailed example.

<!--ts-->
   * [Error classes](#error-classes)
      * [attribute-error](#attribute-error)
      * [bad-concrete-type](#bad-concrete-type)
      * [bad-function-defaults](#bad-function-defaults)
      * [bad-return-type](#bad-return-type)
      * [bad-slots](#bad-slots)
      * [bad-unpacking](#bad-unpacking)
      * [base-class-error](#base-class-error)
      * [duplicate-keyword-argument](#duplicate-keyword-argument)
      * [ignored-abstractmethod](#ignored-abstractmethod)
      * [ignored-type-comment](#ignored-type-comment)
      * [import-error](#import-error)
      * [invalid-annotation](#invalid-annotation)
      * [invalid-directive](#invalid-directive)
      * [invalid-function-type-comment](#invalid-function-type-comment)
      * [invalid-namedtuple-arg](#invalid-namedtuple-arg)
      * [invalid-super-call](#invalid-super-call)
      * [invalid-type-comment](#invalid-type-comment)
      * [invalid-typevar](#invalid-typevar)
      * [key-error](#key-error)
      * [late-directive](#late-directive)
      * [missing-parameter](#missing-parameter)
      * [module-attr](#module-attr)
      * [mro-error](#mro-error)
      * [name-error](#name-error)
      * [not-callable](#not-callable)
      * [not-indexable](#not-indexable)
      * [not-instantiable](#not-instantiable)
      * [not-supported-yet](#not-supported-yet)
      * [not-writable](#not-writable)
      * [pyi-error](#pyi-error)
      * [python-compiler-error](#python-compiler-error)
      * [recursion-error](#recursion-error)
      * [redundant-function-type-comment](#redundant-function-type-comment)
      * [reveal-type](#reveal-type)
      * [unbound-type-param](#unbound-type-param)
      * [unsupported-operands](#unsupported-operands)
      * [wrong-arg-count](#wrong-arg-count)
      * [wrong-arg-types](#wrong-arg-types)
      * [wrong-keyword-args](#wrong-keyword-args)

<!-- Added by: rechen, at: 2019-06-13T12:09-07:00 -->

<!--te-->

## attribute-error

The attribute being accessed may not exist. Often, the reason is that the
attribute is declared in a method other than `__new__` or `__init__`:

<!-- bad -->
```python
class A(object):
  def make_foo(self):
    self.foo = 42
  def consume_foo(self):
    return self.foo  # attribute-error
```

To make pytype aware of `foo`, declare it as a class attribute (with the literal
ellipses) and supply the type in a type comment:

<!-- good -->
```python
class A(object):
  foo = ...  # type: int
```

## bad-concrete-type

A generic type was instantiated with incorrect concrete types.
Example:

<!-- bad -->
```python
from typing import Generic, TypeVar

T = TypeVar('T', int, float)

class A(Generic[T]):
  pass

obj = A[str]()  # bad-concrete-type
```

## bad-function-defaults

An attempt was made to set the `__defaults__` attribute of a function with an
object that is not a constant tuple. Example:

<!-- bad -->
```python
import collections
X = collections.namedtuple("X", "a")
X.__new__.__defaults__ = [None]  # bad-function-defaults
```

## bad-return-type

At least one of the possible types for the return value does not match the
declared return type. Example:

<!-- bad -->
```python
def f(x) -> int:
  if x:
    return 42
  else:
    return None  # bad-return-type
```

## bad-slots

An attempt was made to set the `__slots__` attribute of a class using an object
that's not a string.

<!-- bad -->
```python
class Foo(object):
  __slots__ = (1, 2, 3)
```

## bad-unpacking

A tuple was unpacked into the wrong number of variables. Example:

<!-- bad -->
```python
a, b = (1, 2, 3)  # bad-unpacking
```

## base-class-error

The class definition uses an illegal value for a base class. Example:

<!-- bad -->
```python
class A(42):  # base-class-error
  pass
```

## duplicate-keyword-argument

A positional argument was supplied again as a keyword argument. Example:

<!-- bad -->
```python
def f(x):
  pass
f(True, x=False)  # duplicate-keyword-argument
```

If you believe you are seeing this error due to a bug on pytype's end, see
[this section][pyi-stub-files] for where the type information we use is located.

## ignored-abstractmethod

The abc.abstractmethod decorator was used in a non-abstract class. Example:

<!-- bad -->
```python
import abc
class A(object):  # ignored-abstractmethod
  @abc.abstractmethod
  def f(self):
    pass
```

Add the `abc.ABCMeta` metaclass to fix this issue:

<!-- good -->
```python
import abc
class A(object):
  __metaclass__ = abc.ABCMeta
  @abc.abstractmethod
  def f(self):
    pass
```

## ignored-type-comment

A type comment was found on a line on which type comments are not allowed.
Example:

<!-- bad -->
```python
def f():
  return 42  # type: float  # ignored-type-comment
```

## import-error

The module being imported was not found.

## invalid-annotation

Something is wrong with this annotation. Examples:

<!-- bad -->
```python
from typing import List, TypeVar, Union

T = TypeVar("T")
condition = ...  # type: bool
class _Foo: ...
def Foo():
  return _Foo()

def f(x: List[int, str]):  # bad: too many parameters for List
  pass
def f(x: T):  # bad: the TypeVar appears only once in the signature
  pass
def f(x: Foo):  # bad: not a type
  pass
def f(x: Union):  # bad: no options in the union
  pass
def f(x: int if condition else str):  # bad: ambiguous type
  pass
```

You will also see this error if you use a forward reference in typing.cast or
pass a bad type to `attr.ib`:

<!-- bad -->
```python
import attr
import typing

v = typing.cast("A", None)  # invalid-annotation
class A(object):
  pass

@attr.s
class Foo(object):
  v = attr.ib(type=zip)  # invalid-annotation
```

The solutions are to use a type comment and to fix the type:

<!-- good -->
```python
import attr

v = None  # type: "A"
class A(object):
  pass

@attr.s
class Foo(object):
  v = attr.ib(type=list)
```

## invalid-directive

The error name is misspelled in a pytype disable/enable directive. Example with
a misspelled `name-error`:

<!-- bad -->
```python
x = TypeDefinedAtRuntime  # pytype: disable=nmae-error  # invalid-directive
```

## invalid-function-type-comment

Something was wrong with this function type comment. Examples:

<!-- bad -->
```python
def f(x):
  # type: (int)  # bad: missing return type
  pass
def f(x):
  # type: () -> None  # bad: too few arguments
  pass
def f(x):
  # type: int -> None  # bad: missing parentheses
  pass
```

## invalid-namedtuple-arg

The typename or one of the field names in the namedtuple definition is invalid.
Field names:

*   must not be a Python keyword,
*   must consist of only alphanumeric characters and "_",
*   must not start with "_" or a digit.

Also, there can be no duplicate field names. The typename has the same
requirements, except that it can start with "_".


## invalid-super-call

A call to super without any arguments (Python 3) is being made from an invalid
context. A super call without any arguments should be made from a method or a
function defined within a class. Also, the caller should have at least one
positional argument.

## invalid-type-comment

Something was wrong with this type comment. Examples:

<!-- bad -->
```python
  x = None  # type: NonexistentType  # bad: undefined type
  y = None  # type: int if x else str  # bad: ambiguous type
```

## invalid-typevar

Something was wrong with this TypeVar definition. Examples:

<!-- bad -->
```python
from typing import TypeVar
T = TypeVar("S")  # bad: storing TypeVar "S" as "T"
T = TypeVar(42)  # bad: using a non-str value for the TypeVar name
T = TypeVar("T", str)  # bad: supplying a single constraint
T = TypeVar("T", 0, 100)  # bad: 0 and 100 are not types
```

## key-error

The dictionary key doesn't exist. Example:

<!-- bad -->
```python
x = {}
y = x["y"]  # key-error
```

## late-directive

A `# pytype: disable` without a matching following enable or a `# type: ignore`
appeared on its own line after the first top-level definition. Such a directive
takes effect for the rest of the file, regardless of indentation, which is
probably not what you want:

<!-- bad -->
```python
def f() -> bool:
  # pytype: disable=bad-return-type  # late-directive
  return 42
```

Two equally acceptable fixes:

<!-- good -->
```python
def f() -> bool:
  return 42  # pytype: disable=bad-return-type
```

<!-- good -->
```python
# pytype: disable=bad-return-type
def f() -> bool:
  return 42
# pytype: enable=bad-return-type
```

## missing-parameter

The function was called with a parameter missing. Example:

<!-- bad -->
```python
def add(x, y):
  return x + y
add(42)  # missing-parameter
```

If you believe you are seeing this error due to a bug on pytype's end, see
[this section][pyi-stub-files] for where the type information we use is located.

## module-attr

The module attribute being accessed may not exist. Example:

<!-- bad -->
```python
import sys
sys.nonexistent_attribute  # module-attr
```

## mro-error

A valid method resolution order cannot be created for the class being defined.
Often, the culprit is cyclic inheritance:

<!-- bad -->
```python
class A(object):
  pass
class B(object, A):  # mro-error
  pass
```

## name-error

This name does not exist in the current namespace. Note that types like `List`,
`Dict`, etc., need to be imported from the typing module:

<!-- bad -->
```python
MyListType = List[str]  # name-error
```

<!-- good -->
```python
from typing import List

MyListType = List[str]
```

## not-callable

The object being called or instantiated is not callable. Example:

<!-- bad -->
```python
x = 42
y = x()  # not-callable
```

## not-indexable

The object being indexed is not indexable. Example:

<!-- bad -->
```python
tuple[3]  # not-indexable
```

## not-instantiable

The class cannot be instantiated because it has abstract methods. Example:

<!-- bad -->
```python
import abc
class A(object):
  __metaclass__ = abc.ABCMeta
  @abc.abstractmethod
  def f(self):
    pass
A()  # not-instantiable
```

## not-supported-yet

This feature is not yet supported by pytype.

## not-writable

The object an attribute was set on doesn't have that attribute, or that
attribute isn't writable:

<!-- bad -->
```python
class Foo(object):
  __slots__ = ("x", "y")

Foo().z = 42  # not-writable
```

## pyi-error

The pyi file contains a syntax error.

If you encounter this error in a pyi file that you did not create yourself,
please [file a bug][new-bug].

## python-compiler-error

The Python code contains a syntax error.

## recursion-error

A recursive definition was found in a pyi file. Example:

<!-- bad -->
```python
class A(B): ...
class B(A): ...
```

If you encounter this error in a pyi file that you did not create yourself,
please [file a bug][new-bug].

## redundant-function-type-comment

Using both inline annotations and a type comment to annotate the same function
is not allowed. Example:

<!-- bad -->
```python
def f() -> None:
  # type: () -> None  # redundant-function-type-comment
  pass
```

## reveal-type

The error message displays the type of the expression passed to it. Example:

<!-- good -->
```python
import os
reveal_type(os.path.join("hello", u"world"))  # reveal-type: unicode
```

This feature is implemented as an error to ensure that `reveal_type()` calls are
removed after debugging.

## unbound-type-param

This error currently applies only to pyi files. The type parameter is not bound
to a class or function. Example:

<!-- bad -->
```python
from typing import AnyStr
x = ...  # type: AnyStr  # unbound-type-param
```

Unbound type parameters are meaningless as types. If you want to take advantage
of types specified by a type parameter's constraints or bound, specify those
directly. So the above example should be rewritten as:

<!-- good -->
```python
from typing import Union
x = ...  # type: Union[str,  unicode]
```

## unsupported-operands

A binary operator was called with incompatible arguments. Example:

<!-- bad -->
```python
x = "hello" ^ "world"  # unsupported-operands
```

## wrong-arg-count

The function was called with the wrong number of arguments. Example:

<!-- bad -->
```python
def add(x, y):
  return x + y
add(1, 2, 3)  # wrong-arg-count
```

If you believe you are seeing this error due to a bug on pytype's end, see
[this section][pyi-stub-files] for where the type information we use is located.

## wrong-arg-types

The function was called with the wrong argument types. Example:

<!-- bad -->
```python
def f(x: int):
  pass
f(42.0)  # wrong-arg-types
```

If you believe you are seeing this error due to a bug on pytype's end, see
[this section][pyi-stub-files] for where the type information we use is located.

## wrong-keyword-args

The function was called with the wrong keyword arguments. Example:

<!-- bad -->
```python
def f(x=True):
  pass
f(y=False)  # wrong-keyword-args
```

If you believe you are seeing this error due to a bug on pytype's end, see
[this section][pyi-stub-files] for where the type information we use is located.

<!-- General references -->
[pyi-stub-files]: user_guide.md#pytypes-pyi-stub-files
[silencing-errors]: user_guide.md#silencing-errors

<!-- References with different internal and external versions -->
[new-bug]: https://github.com/google/pytype/issues/new
