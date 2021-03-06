sqlalchemy-json
###############

SQLAlchemy-JSON provides mutation-tracked JSON types to SQLAlchemy_:

* ``MutableJson`` is a straightforward implementation for keeping track of top-level changes to JSON objects;
* ``NestedMutableJson`` is an extension of this which tracks changes even when these happen in nested objects or arrays (Python ``dicts`` and ``lists``).


Examples
========

Basic change tracking
---------------------

This is essentially the SQLAlchemy `mutable JSON recipe`_. We define a simple author model which list the author's name and a property ``handles`` for various social media handles used:

.. code-block:: python

    class Author(Base):
        name = Column(Text)
        handles = Column(MutableJson)

The example below loads one of the existing authors and retrieves the mapping of social media handles. The error in the twitter handle is then corrected and committed. The change is detected by SQLAlchemy and the appropriate ``UPDATE`` statement is generated.

.. code-block:: python

    >>> author = session.query(Author).first()
    >>> author.handles
    {'twitter': '@JohnDoe', 'facebook': 'JohnDoe'}
    >>> author.handles['twitter'] = '@JDoe'
    >>> session.commit()
    >>> author.handles
    {'twitter': '@JDoe', 'facebook': 'JohnDoe'}


Nested change tracking
----------------------

The example below defines a simple model for articles. One of the properties on this model is a mutable JSON structure called ``references`` which includes a count of links that the article contains, grouped by domain:

.. code-block:: python

    class Article(Base):
        author = Column(ForeignKey('author.name'))
        content = Column(Text)
        references = Column(NestedMutableJson)

With this in place, an existing article is loaded and its current references inspected. Following that, the count for one of these is increased by ten, and the session is committed:

.. code-block:: python

    >>> article = session.query(Article).first()
    >>> article.references
    {'github.com': {'edelooff/sqlalchemy-json': 4, 'zzzeek/sqlalchemy': 7}}
    >>> article.references['github.com']['edelooff/sqlalchemy-json'] += 10
    >>> session.commit()
    >>> article.references
    {'github.com': {'edelooff/sqlalchemy-json': 14, 'zzzeek/sqlalchemy': 7}}

Had the articles model used ``MutableJson`` like in the previous example this code would have failed. This is because the top level dictionary is never altered directly. The *nested* mutable ensures the change happening at the lower level *bubbles up* to the outermost container.


Dependencies
============

* ``SQLAlchemy-utils`` for its existing and database-engine specific choice of either native or simulated JSON type;
* ``six`` to support both Python 2 and 3.


Changelog
=========

0.2.1
-----

* Fixes a typo in the README found after uploading 0.2.0 to PyPI.


0.2.0 (unreleased)
------------------

* Now uses ``JSONType`` provided by SQLAlchemy-utils_ to handle backend storage;
* **Backwards incompatible**: Changed class name ``JsonObject`` to ``MutableJson`` and ``NestedJsonObject`` to ``NestedMutableJson``
* Outermost container for ``NestedMutableJson`` can now be an ``array`` (Python ``list``)


0.1.0 (unreleased)
------------------

Initial version. This initially carried a 1.0.0 version number but has never been released on PyPI.


.. _mutable json recipe: http://docs.sqlalchemy.org/en/latest/core/custom_types.html#marshal-json-strings
.. _sqlalchemy: https://www.sqlalchemy.org/
.. _sqlalchemy-utils: https://sqlalchemy-utils.readthedocs.io/

