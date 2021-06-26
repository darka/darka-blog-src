---
title: "Python Enumeration Comparisons"
description: "Working around somewhat unintuitive Enum comparison behaviour in Python"
date: 2021-06-26T12:30:00Z
draft: false
summary: Working around somewhat unintuitive Enum comparison behaviour in Python
tags: ["python"]
---

# Example

Imagine we need to store a mapping between animals and their corresponding animal classes inside an enumeration.
We start by defining an animal class enum:

{{< highlight python >}}
>>> from enum import Enum
>>> class AnimalClass(Enum):
...    MAMMAL = 1
...    BIRD = 2
...    REPTILE = 3
{{< /highlight >}}

We then define our animal enum:
{{< highlight python >}}
>>> class Animal(Enum):
...    MAGPIE = AnimalClass.BIRD
...    SQUIRREL = AnimalClass.MAMMAL
...    OPOSSUM = AnimalClass.MAMMAL
{{< /highlight >}}

However, if we try to compare members of the `Animal` enum, the behaviour may be somewhat unintuitive:
{{< highlight python >}}
>>> Animal.MAGPIE == Animal.OPOSSUM
False
>>> Animal.SQUIRREL == Animal.OPOSSUM
True
{{< /highlight >}}

Enum members are compared by identity, so only the value matters, and hence `SQUIRREL` and `OPOSSUM` evaluate as equal.

# Comparisons

Let's examine the enum comparison behaviour further by taking another simple example:

{{< highlight python >}}
>>> class MyEnum(Enum):
...     FOO = 1
...     BAR = 2
...
>>> MyEnum.FOO.value == MyEnum.BAR.value
False
>>> MyEnum.FOO == MyEnum.BAR
False
{{< /highlight >}}

So far so good. The values are different, hence comparison between members with different values evaluates to `False`.

Let's add another member to the enumeration, but give it the same value:

{{< highlight python >}}
>>> class MyEnum(Enum):
...     FOO = 1
...     BAR = 2
...     BAZ = 2
...
>>> MyEnum.BAR.value == MyEnum.BAZ.value
True
>>> MyEnum.BAR == MyEnum.BAZ
True
{{< /highlight >}}

As the values of enumeration members are the same, the members themselves evaluate as equal!

> __Note:__ We can disallow defining enumerations with duplicate values by using [the `@unique` decorator][unique-decorator]

What about comparing enumeration members with their own values?

{{< highlight python >}}
>>> MyEnum.BAR.value == 2
True
>>> MyEnum.BAR == 2
False
{{< /highlight >}}

Although we can compare the values directly, comparing members and their values evaluates to `False`.

One way to avoid issues with comparisons, is to use automatic values:

{{< highlight python >}}
>>> from enum import Enum, auto
>>> class MyEnum(Enum):
...     FOO = auto()
...     BAR = auto()
...     BAZ = auto()
...
>>> MyEnum.BAR.value == MyEnum.BAZ.value
False
>>> MyEnum.BAR == MyEnum.BAZ
False
{{< /highlight >}}

This ensures all values are different, but means we can no longer store anything useful as a value.
In some cases (as in the original example) we may want to keep the values around rather than replace them with `auto()`.
If some of the values are the same, the equality behaviour described above may not be what we want.

There is a way to work around this by storing the value as extra attribute on an enumeration.

# Extra Attributes

Python supports adding extra attributes to an enumeration by [overriding the `__new__` method:][custom-new]

{{< highlight python >}}
>>> class MyEnum(Enum):
...    FOO = 1, 'a'
...    BAR = 2, 'b'
...    BAZ = 3, 'b'
...
...    def __new__(cls, value, extra):
...        obj = object.__new__(cls)
...        obj._value_ = value
...        obj.extra = extra
...        return obj
{{< /highlight >}}

We can then access the new attribute on any member:

{{< highlight python >}}
>>> MyEnum.FOO.value
1
>>> MyEnum.FOO.extra
'a'
{{< /highlight >}}

This allows distinct enum members to share the same attribute value.

{{< highlight python >}}
>>> MyEnum.BAR.extra == MyEnum.BAZ.extra
True
>>> MyEnum.BAR.value == MyEnum.BAZ.value
False
>>> MyEnum.BAR == MyEnum.BAZ
False
{{< /highlight >}}

Hence, we can now store a mapping inside an enumeration with members sharing the same attribute value, but also ensure
they do not evaluate as equal.

# Fixing the Example

We start by defining our `AnimalClass` enumeration again (this time using `auto()`):
{{< highlight python >}}
>>> class AnimalClass(Enum):
...    MAMMAL = auto()
...    BIRD = auto()
...    REPTILE = auto()
{{< /highlight >}}

We then define our `Animal` enum, but this time we override `__new__` and store the animal class as an extra attribute:

{{< highlight python >}}
>>> class Animal(Enum):
...     def __new__(cls, value, animal_class):
...         obj = object.__new__(cls)
...         obj._value_ = value
...         obj.animal_class = animal_class
...         return obj
...     MAGPIE = (auto(), AnimalClass.BIRD)
...     SQUIRREL = (auto(), AnimalClass.MAMMAL)
...     OPOSSUM = (auto(), AnimalClass.MAMMAL)
{{< /highlight >}}

This gives us the behaviour we want, where the enumeration members do not evaluate as equal, although the animal class they belong to is the same:

{{< highlight python >}}
>>> Animal.SQUIRREL == Animal.OPOSSUM
False
>>> Animal.SQUIRREL.animal_class == Animal.OPOSSUM.animal_class
True
{{< /highlight >}}

# Disclaimer

Although the enumeration now behaves how we want, the readability of the code has been significantly reduced. Someone who is not familiar with these
enumeration tricks will find it difficult to decipher what the fixed example does, and will likely be inclined to convert it to a
simple enum, potentially introducing bugs in the process.

Caution must be taken when introducing a complicated piece of code to a codebase.

[unique-decorator]: https://docs.python.org/3/library/enum.html#enum.unique
[custom-new]: https://docs.python.org/3/library/enum.html#using-a-custom-new
