===========
namedlist
===========

WARNING
=======

This package is no longer maintained, except for exceptional cases.
Please use the built-in dataclasses module instead.

Overview
========

namedlist provides 2 factory functions, namedlist.namedlist and
namedlist.namedtuple. namedlist.namedtuple is similar to
collections.namedtuple, with the following differences:

* namedlist.namedtuple supports per-field default values.

* namedlist.namedtuple supports an optional default value, to be used
  by all fields that do not have an explicit default value.


namedlist.namedlist is similar, with this additional difference:

* namedlist.namedlist instances are mutable.

Typical usage
=============

You can use namedlist like a mutable namedtuple::

    >>> from namedlist import namedlist

    >>> Point = namedlist('Point', 'x y')
    >>> p = Point(1, 3)
    >>> p.x = 2
    >>> assert p.x == 2
    >>> assert p.y == 3

Or, you can specify a default value for all fields::

    >>> Point = namedlist('Point', 'x y', default=3)
    >>> p = Point(y=2)
    >>> assert p.x == 3
    >>> assert p.y == 2

Or, you can specify per-field default values::

    >>> Point = namedlist('Point', [('x', 0), ('y', 100)])
    >>> p = Point()
    >>> assert p.x == 0
    >>> assert p.y == 100

You can also specify the per-field defaults with a mapping, instead
of an iterable. Note that this is only useful with an ordered
mapping, such as an OrderedDict::

    >>> from collections import OrderedDict
    >>> Point = namedlist('Point', OrderedDict((('y', 0),
    ...                                         ('x', 100))))
    >>> p = Point()
    >>> p
    Point(y=0, x=100)

The default value will only be used if it is provided and a per-field
default is not used::

    >>> Point = namedlist('Point', ['x', ('y', 100)], default=10)
    >>> p = Point()
    >>> assert p.x == 10
    >>> assert p.y == 100

If you use a mapping, the value NO_DEFAULT is convenient to specify
that a field uses the default value::

    >>> from namedlist import NO_DEFAULT
    >>> Point = namedlist('Point', OrderedDict((('y', NO_DEFAULT),
    ...                                         ('x', 100))),
    ...                            default=5)
    >>> p = Point()
    >>> assert p.x == 100
    >>> assert p.y == 5

namedtuple is similar, except the instances are immutable::

    >>> from namedlist import namedtuple
    >>> Point = namedtuple('Point', 'x y', default=3)
    >>> p = Point(y=2)
    >>> assert p.x == 3
    >>> assert p.y == 2
    >>> p.x = 10
    Traceback (most recent call last):
    ...
    AttributeError: can't set attribute

All of the documentation below in the Specifying Fields and Specifying
Defaults sections applies to namedlist.namedlist and
namedlist.namedtuple.

Creating types
==============

Specifying Fields
-----------------

Fields in namedlist.namedlist or namedlist.namedtuple can be specified
as in collections.namedtuple: as either a string specifing the field
names, or as a iterable of field names. These two uses are
equivalent::

    >>> Point = namedlist('Point', 'x y')
    >>> Point = namedlist('Point', ['x', 'y'])

As are these::

    >>> Point = namedtuple('Point', 'x y')
    >>> Point = namedtuple('Point', ['x', 'y'])

If using a string, commas are first converted to spaces. So these are
equivalent::

    >>> Point = namedlist('Point', 'x y')
    >>> Point = namedlist('Point', 'x,y')


Specifying Defaults
-------------------

Per-field defaults can be specified by supplying a 2-tuple (name,
default_value) instead of just a string for the field name. This is
only supported when you specify a list of field names::

    >>> Point = namedlist('Point', [('x', 0), ('y', 0)])
    >>> p = Point(3)
    >>> assert p.x == 3
    >>> assert p.y == 0

Or, using namedtuple::

    >>> Point = namedtuple('Point', [('x', 0), ('y', 0)])
    >>> p = Point(3)
    >>> assert p.x == 3
    >>> assert p.y == 0


In addition to, or instead of, these per-field defaults, you can also
specify a default value which is used when no per-field default value
is specified::

    >>> Point = namedlist('Point', 'x y z', default=0)
    >>> p = Point(y=3)
    >>> assert p.x == 0
    >>> assert p.y == 3
    >>> assert p.z == 0

    >>> Point = namedlist('Point', [('x', 0), 'y', ('z', 0)], default=4)
    >>> p = Point(z=2)
    >>> assert p.x == 0
    >>> assert p.y == 4
    >>> assert p.z == 2

In addition to supplying the field names as an iterable of 2-tuples,
you can also specify a mapping. The keys will be the field names, and
the values will be the per-field default values. This is most useful
with an OrderedDict, as the order of the fields will then be
deterministic.  The module variable NO_DEFAULT can be specified if you
want a field to use the per-type default value instead of specifying
it with a field::

    >>> Point = namedlist('Point', OrderedDict((('x', 0),
    ...                                         ('y', NO_DEFAULT),
    ...                                         ('z', 0),
    ...                                         )),
    ...                            default=4)
    >>> p = Point(z=2)
    >>> assert p.x == 0
    >>> assert p.y == 4
    >>> assert p.z == 2

Writing to values
-----------------

Instances of the classes generated by namedlist.namedlist are fully
writable, unlike the tuple-derived classes returned by
collections.namedtuple or namedlist.namedtuple::

    >>> Point = namedlist('Point', 'x y')
    >>> p = Point(1, 2)
    >>> p.y = 4
    >>> assert p.x == 1
    >>> assert p.y == 4


Specifying __slots__
--------------------

For namedlist.namedlist, by default, the returned class sets __slots__
which is initialized to the field names. While this decreases memory
usage by eliminating the instance dict, it also means that you cannot
create new instance members.

To change this behavior, specify use_slots=False when creating the
namedlist::

    >>> Point = namedlist('Point', 'x y', use_slots=False)
    >>> p = Point(0, 1)
    >>> p.z = 2
    >>> assert p.x == 0
    >>> assert p.y == 1
    >>> assert p.z == 2

However, note that this method does not add the new variable to
_fields, so it is not recognized when iterating over the instance::

    >>> list(p)
    [0, 1]
    >>> sorted(p._asdict().items())
    [('x', 0), ('y', 1)]


Additional class members
------------------------

namedlist.namedlist and namedlist.namedtuple classes contain these members:

* _asdict(): Returns a dict which maps field names to their
  corresponding values.

* _fields: Tuple of strings listing the field names. Useful for introspection.


Renaming invalid field names
----------------------------

This functionality is identical to collections.namedtuple. If you
specify rename=True, then any invalid field names are changed to _0,
_1, etc. Reasons for a field name to be invalid are:

* Zero length strings.

* Containing characters other than alphanumerics and underscores.

* A conflict with a Python reserved identifier.

* Beginning with a digit.

* Beginning with an underscore.

* Using the same field name more than once.

For example::

    >>> Point = namedlist('Point', 'x x for', rename=True)
    >>> assert Point._fields == ('x', '_1', '_2')


Mutable default values
----------------------

For namedlist.namelist, be aware of specifying mutable default
values. Due to the way Python handles default values, each instance of
a namedlist will share the default. This is especially problematic
with default values that are lists. For example::

    >>> A = namedlist('A', [('x', [])])
    >>> a = A()
    >>> a.x.append(4)
    >>> b = A()
    >>> assert b.x == [4]

This is probably not the desired behavior, so see the next section.


Specifying a factory function for default values
------------------------------------------------

For namedlist.namedlist, you can supply a zero-argument callable for a
default, by wrapping it in a FACTORY call. The only change in this
example is to change the default from `[]` to `FACTORY(list)`. But
note that `b.x` is a new list object, not shared with `a.x`::

    >>> from namedlist import FACTORY
    >>> A = namedlist('A', [('x', FACTORY(list))])
    >>> a = A()
    >>> a.x.append(4)
    >>> b = A()
    >>> assert b.x == []

Every time a new instance is created, your callable (in this case,
`list`), will be called to produce a new instance for the default
value.

Iterating over instances
------------------------

Because instances are iterable (like lists or tuples), iteration works
the same way. Values are returned in definition order::

    >>> Point = namedlist('Point', 'x y z t')
    >>> p = Point(1.0, 42.0, 3.14, 2.71828)
    >>> for value in p:
    ...    print(value)
    1.0
    42.0
    3.14
    2.71828


namedlist specific functions
============================

_update
-------

`namedlist._update()` is similar to `dict.update()`. It is used to
mutate a namedlist.namedlist instance with new values::

    >>> Point = namedlist('Point', 'x y z')
    >>> p = Point(1, 2, 3)
    >>> p.z = 4
    >>> p._update(y=5, x=6)
    >>> p
    Point(x=6, y=5, z=4)

    >>> p._update({'x': 7, 'z': 8})
    >>> p
    Point(x=7, y=5, z=8)

    >>> p._update([('z', 9), ('y', 10)])
    >>> p
    Point(x=7, y=10, z=9)


Creating and using instances
============================

Because the type returned by namedlist or namedtuple is a normal
Python class, you create instances as you would with any Python class.

Bitbucket vs. GitLab
====================

The repository used to be on Bitbucket in Mercurial format.  But
Bitbucket dropped Mercurial support and did not provide any way to
migrate issues to a git repository, even one hosted on Bitbucket.  So,
I abandoned Bitbucket and moved the code to GitLab.  Thus, all of the
issue were lost, and new issues started again with #1.  I'm naming the
GitLab issues as #GH-xx, and the old Bitbucket issues as #BB-xx.  I'm
still angry at Bitbucket for forcing this change.
