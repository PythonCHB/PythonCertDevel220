###############################
Part 3: Iterators and Iterables
###############################

Iterators and Iterables are key language features of Python 3, that help
to make code easy to write and maintain.

First, we are going to look at some background from Python 2, before moving on
to cover the main topic.

Background
==========

Python used to be all about sequences --- a good chunk of anything you
did was stored in a sequence or involved manipulating one.

-  lists

-  tuples

-  strings

-  dict.keys()

-  dict.values()

-  dict.items()

-  zip()


In **Python2** those are all sequences.  It turns out, however, that
the most common operation for sequences is to iterate through them:

::

        for item in a_sequence:
            do_something_with_item

So fairly early in Python2, Python introduced the idea of the
"iterable".  An iterable is something you can, well, iterate over in a
for loop, but often does not keep the whole sequence in memory at
once.  After all, why make a copy of something just to look at all its
items?

For example, in python2: ``dict.keys()`` returns a list of all the
keys in the dict.  But why make a full copy of all the keys, when all
you want to do is:

::

     for key in dict.keys():
         do_something_with(key)

Even worse ``dict.items()`` created a full list of
``(key,value)`` tuples -- a complete copy of all the data in the
dict.
Even worse ``enumerate(dict.items())`` created a whole list of
``(index, (key, value))`` tuples -- lots of copies of everything.

Python2 then introduced "iterable" versions of a number of functions
and methods:


| ``itertools.izip``
| ``dict.iteritems()``
| ``dict.iterkeys()``
| ``dict.itervalues()``


So you could now iterate through those things without copying anything.

Python 3
--------


**Python3** embraces iterables --- now everything that can be an
iterator is already an iterator --- no unnecessary copies.  An
iterator is an iterable that has been made more efficient by removing
as much from memory as possible. Therefore, if you need a list, you
have to make the list explicitly, as in::

    list(dict.keys())


Also, there is an entire module: ``itertools`` that provides nifty
ways to iterate through various iterables.  So, while we used to think in terms of sequences, we can now think in terms of iterables.

Iterators and Iterables
=======================

Let's see an example of how Iteration makes Python code so readable:

.. code-block:: python

     for x in just_about_anything:
         do_stuff(x)

An *iterable* is anything that can be looped over sequentially, so it
does not have to be a typical "sequence" type: list, tuple, etc.
For example, a string is iterable (iterate over the characters).
So is a set.

An *iterator* is an iterable that remembers state. All sequences are
iterable, but not all sequences are iterators. To make a sequence an
iterator, you can call it with iter::

    my_iter = iter(my_sequence)

The distiction between `iterables` and `iterators` is a bit subtle:

An *iterable* is something that you can iterate over -- that is, put in a for loop.

An *iterator* is the object that actually manges the iteration -- it keeps track of how which items have been used, and delivers them when asked for.

The ``for`` statement usually takes care of managing all that, but you can do it bit by bit by hand as well.


The Iteration Protocol
----------------------

Python defines a protocol for mamking and working with iterables, Any object that conforms to that protocol can be used in a for loop, and other contexts in which an iterable is expected (the list constuctor, for instance).

The easiest way to make an object iterable, is to implement the
``__getitem__`` method.

.. code-block:: python

     class T:
         def __getitem__(self, position):
             if position > 5:
              return position

``iter()``
----------

Part of this protocol is the ``iter()`` function -- it takes any iterable as an argument, and returs an itertor -- primed and ready to be used.

It first looks for the __iter__() method, and if none is found, uses get_item to create the iterator.
The ``iter()`` function in action:

.. code-block:: ipython

     In []: iter([2,3,4])
     Out[]: <listiterator at 0x101e01350>
     In []: iter("a string")
     Out[]: <iterator at 0x101e01090>
     In []: iter( ('a', 'tuple') )
     Out[]: <tupleiterator at 0x101e01710>

``next()``
----------

So how do you get the items from an iterator?

Another key part of the iterator protocol is the ``next()`` function. When passed in a iterator, it returns the next item:

.. code-block:: ipython

     In []: a_list = [1,2,3]

     # first get the iterator from the list
     In []: list_iter = iter(a_list)

     # then ask for its items, one by one:
     In []: next(list_iter)
     Out[]: 1

     In []: next(list_iter)
     Out[]: 2

     In []: next(list_iter)
     Out[]: 3

     # what happens when there are no more?
     In []: next(list_iter)
     --------------------------------------------------
     StopIteration     Traceback (most recent call last)
     <ipython-input-15-1a7db9b70878> in <module>()
     ----> 1 next(list_iter)
     StopIteration:

As you can see, when you call ``next()`` when there are no more items (the iterator is "exhausted"), a ``StopIteration`` exception is raised, indicating that you are done.


Use Iterators When You Can
==========================

Consider the example from the trigrams problem:
(http://codekata.com/kata/kata14-tom-swift-under-the-milkwood/)

You have a list of words and you want to go through it, three at a
time, and match up pairs with the following word.
The *non-pythonic* way to do that is to loop through the indices:

.. code-block:: python

     for i in range(len(words)-2):
         triple = words[i:i+3]

It works, and is fairly efficient, but what about::


    for triple in zip(words[:-2], words[1:-1], words[2:-2]):

zip() returns an *iterable* --- it does not build up the whole list, so
this is quite efficient.  However, we are still slicing: (``[1:]``), which
produces a copy --- so we are creating three copies of the list ---
not so good if memory is tight.  Note that they are shallow copies, so
this is not terribly bad.  Nevertheless, we can do better.

The ``itertools`` module has a ``islice()`` (iterable slice)
function.  It returns an iterator over a slice of a sequence --- so no
more copies:

.. code-block:: python

     from itertools import islice
     triplets = zip(words, islice(words, 1, None), islice(words, 2, None))
     for triplet in triplets:
         print(triplet)
     ('this', 'that', 'the')
     ('that', 'the', 'other')
     ('the', 'other', 'and')
     ('other', 'and', 'one')
     ('and', 'one', 'more')


The Iterator Protocol
---------------------

There are two perspectives to the iterator protocol:

1) Working with an iterable:

You use ``iter()`` to get an iterator, and  ``next()`` to get the next item in the iterable.

2) Making a custom iterable:

An iterable must have an ``__iter__()`` method that returns an iterator.

An iterator must have a ``__next__()`` method that returns the next item,
and raises ``StopIteration`` when there are no more items.


The main thing that differentiates an iterator from an iterable (sequence) is that an iterator saves state -- it keeps track of which items have been already used, and which are left.

The ``__iter__()`` method
.........................

A class' ``__iter()`` method needs to return an iterator, ready to have ``__next__()`` called.


Often a custom iterable will return the iterator object itself.

The main thing that differentiates an iterator from an iterable (sequence or other object) is that an iterator saves state -- it keeps track of which items have been already used, and which are left.

If an iterable returns itself, than the ``__iter__()`` method is the place to initialize (or reset) the counter on where the iteration has been so far.


The ``__next__()`` method
.........................

The ``__next__()`` method returns the next item from the container (or other source).
If there are no further items it raises the ``StopIteration`` exception.

Probably the best way to understand this is with an example.

Making an Iterable
------------------

The ``range()`` builtin object is an iterable that produces a range of numbers, as they are asked for -- it does not create the sequence of number ahead of time, but rather, creates each one as it is needed.

.. note:: In Python 2, ``range()`` *did* create the full list as soon as you called it. As this was pretty inefficient, an ``xrange()`` function was created that generated the numbers on the fly. In python 3, the built in range() no longer creates a list, so ``xrange()`` is no longer needed. You may still see ``xrange()`` in old Python 2 code -- change it to range() when porting to Python 3.


You can create a simple version of ``range()`` like this:

.. code-block:: python

        class My_range:
            def __init__(self, stop=5):
                self.current = 0
                self.stop = stop
            def __iter__(self):
                return self
            def __next__(self):
                if self.current < self.stop:
                    self.current += 1
                    return self.current
                else:
                    raise StopIteration


What does ``for`` do?
---------------------

Now that we know the iterator protocol, we can write something like a
for loop:

(:download:`my_for.py  <../examples/iterators_generators/my_for.py>`)

.. code-block:: python

     def my_for(an_iterable, func):
         """
         Emulation of a for loop.
         func() will be called with each item in an_iterable
        
         equiv of "for i in l: func(i)"
         """

         iterator = iter(an_iterable)
         while True:
             try:
                 i = next(iterator)
             except StopIteration:
                 break
             func(i)

Summary
-------

Iterators and Iterables are fundamental concepts in Python. Although the language
can be confusing, the underlying concepts are quite straightforward.

In the lesson assignment you will have opportunities to practice and apply using them.
