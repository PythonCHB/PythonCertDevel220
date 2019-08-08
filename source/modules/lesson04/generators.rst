##################
Part 4: Generators
##################

Generators are a special kind of object that is like a function that can be paused while preserving state, and then be resumed.

They are designed to work with the iterator protocol, so they can easily make iterables that "generate" values on the fly, rather than those stored in a sequence. This was the original use case, hence the name. But generators can be used in other places where it is handy to pause a function while maintaining state. They are used in pytest fixtures, for example, and in asynchronous programming -- both advanced topics for a later date.

But for now, we will focus on their use for making iterators.

Conceptually, iterators are about various ways to loop over data. They can iterate over a sequence of data that already exists, or they generate values on the fly.

In general, you can use either a custom class a generator function to make an iterator -- in fact, a generator is a type of iterator. BUt using a generator function is often easier -- it does some of the book-keeping for you and therefore involve simpler code.

 
Generator Functions
===================

Generator functions are a special kind of function that returns a generator when called, rather than a simple value.

``yield``
---------

To make a generator function, you write it like a regular function, except that you use ``yield`` instead of ``return``. Any function with a ``yield`` statement in it is a generator function.

For example:


.. code-block:: python

        def a_generator_function(params):
            some_stuff
            yield something

Generators "yield" a value, rather than returning a value. 
It *does* "return" a value, but rather than ending execution of the
function, it preserves function state so that it can pick up where it
left off.  In other words, state is preserved between ``yields`` statements.

A function with ``yield`` in it is a factory for a generator, or "generator function". 
Each time you call it, you get a new generator:

::

        gen_a = a_generator()
        gen_b = a_generator()

Each instance keeps its own state.

To master yield, you must understand that when you call the function,
the code you have written in the function body does not run.  The
function only returns the generator object.  The actual code in the
function is run when next() is called on the generator itself.

An example: an implementation of range() as a generator:

::

        def y_range(start, stop, step=1):
            i = start
            while i < stop:
                yield i
                i += step

Generator Comprehensions
........................

Generator Comprehensions are yet another way to make a generator, with the comprehension syntax:

.. code-block:: python


        >>> [x * 2 for x in [1, 2, 3]]
        [2, 4, 6]

        >>> (x * 2 for x in [1, 2, 3])
        <generator object <genexpr> at 0x10911bf50>

        >>> for n in (x * 2 for x in [1, 2, 3]):
        ...   print n
        ... 2 4 6

They are the same as list comprehension, except that they don't run through teh whle loop and make a list, but rather, make a generator that will loop through the items when called by the iterator protocol.

