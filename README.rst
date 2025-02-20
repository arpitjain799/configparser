.. image:: https://img.shields.io/pypi/v/configparser.svg
   :target: https://pypi.org/project/configparser

.. image:: https://img.shields.io/pypi/pyversions/configparser.svg

.. image:: https://github.com/jaraco/configparser/workflows/tests/badge.svg
   :target: https://github.com/jaraco/configparser/actions?query=workflow%3A%22tests%22
   :alt: tests

.. image:: https://img.shields.io/badge/code%20style-black-000000.svg
   :target: https://github.com/psf/black
   :alt: Code style: Black

.. image:: https://readthedocs.org/projects/configparser/badge/?version=latest
   :target: https://configparser.readthedocs.io/en/latest/?badge=latest

.. image:: https://img.shields.io/badge/skeleton-2023-informational
   :target: https://blog.jaraco.com/skeleton

.. image:: https://tidelift.com/badges/package/pypi/configparser
   :target: https://tidelift.com/subscription/pkg/pypi-configparser?utm_source=pypi-configparser&utm_medium=readme


This package is a backport of the refreshed and enhanced ConfigParser from
later Python versions. To use the backport instead of the built-in version,
simply import it explicitly as a backport::

  from backports import configparser

To use the backport on Python 2 and the built-in version on
Python 3, use the standard invocation::

  import configparser

For detailed documentation consult the vanilla version at
http://docs.python.org/3/library/configparser.html.

Why you'll love ``configparser``
--------------------------------

Whereas almost completely compatible with its older brother, ``configparser``
sports a bunch of interesting new features:

* full mapping protocol access (`more info
  <http://docs.python.org/3/library/configparser.html#mapping-protocol-access>`_)::

    >>> parser = ConfigParser()
    >>> parser.read_string("""
    [DEFAULT]
    location = upper left
    visible = yes
    editable = no
    color = blue

    [main]
    title = Main Menu
    color = green

    [options]
    title = Options
    """)
    >>> parser['main']['color']
    'green'
    >>> parser['main']['editable']
    'no'
    >>> section = parser['options']
    >>> section['title']
    'Options'
    >>> section['title'] = 'Options (editable: %(editable)s)'
    >>> section['title']
    'Options (editable: no)'

* there's now one default ``ConfigParser`` class, which basically is the old
  ``SafeConfigParser`` with a bunch of tweaks which make it more predictable for
  users. Don't need interpolation? Simply use
  ``ConfigParser(interpolation=None)``, no need to use a distinct
  ``RawConfigParser`` anymore.

* the parser is highly `customizable upon instantiation
  <http://docs.python.org/3/library/configparser.html#customizing-parser-behaviour>`__
  supporting things like changing option delimiters, comment characters, the
  name of the DEFAULT section, the interpolation syntax, etc.

* you can easily create your own interpolation syntax but there are two powerful
  implementations built-in (`more info
  <http://docs.python.org/3/library/configparser.html#interpolation-of-values>`__):

  * the classic ``%(string-like)s`` syntax (called ``BasicInterpolation``)

  * a new ``${buildout:like}`` syntax (called ``ExtendedInterpolation``)

* fallback values may be specified in getters (`more info
  <http://docs.python.org/3/library/configparser.html#fallback-values>`__)::

    >>> config.get('closet', 'monster',
    ...            fallback='No such things as monsters')
    'No such things as monsters'

* ``ConfigParser`` objects can now read data directly `from strings
  <http://docs.python.org/3/library/configparser.html#configparser.ConfigParser.read_string>`__
  and `from dictionaries
  <http://docs.python.org/3/library/configparser.html#configparser.ConfigParser.read_dict>`__.
  That means importing configuration from JSON or specifying default values for
  the whole configuration (multiple sections) is now a single line of code. Same
  goes for copying data from another ``ConfigParser`` instance, thanks to its
  mapping protocol support.

* many smaller tweaks, updates and fixes

A few words about Unicode
-------------------------

``configparser`` comes from Python 3 and as such it works well with Unicode.
The library is generally cleaned up in terms of internal data storage and
reading/writing files.  There are a couple of incompatibilities with the old
``ConfigParser`` due to that. However, the work required to migrate is well
worth it as it shows the issues that would likely come up during migration of
your project to Python 3.

The design assumes that Unicode strings are used whenever possible [1]_.  That
gives you the certainty that what's stored in a configuration object is text.
Once your configuration is read, the rest of your application doesn't have to
deal with encoding issues. All you have is text [2]_. The only two phases when
you should explicitly state encoding is when you either read from an external
source (e.g. a file) or write back.

Versioning
----------

This project uses `semver <https://semver.org/spec/v2.0.0.html>`_ to
communicate the impact of various releases while periodically syncing
with the upstream implementation in CPython.
The `history <https://configparser.readthedocs.io/en/latest/history.html>`_
serves as a reference indicating which versions incorporate
which upstream functionality.

Prior to the ``4.0.0`` release, `another scheme
<https://github.com/jaraco/configparser/blob/3.8.1/README.rst#versioning>`_
was used to associate the CPython and backports releases.

Maintenance
-----------

This backport was originally authored by Łukasz Langa, the current vanilla
``configparser`` maintainer for CPython and is currently maintained by
Jason R. Coombs:

* `configparser repository <https://github.com/jaraco/configparser>`_

* `configparser issue tracker <https://github.com/jaraco/configparser/issues>`_

For Enterprise
==============

Available as part of the Tidelift Subscription.

This project and the maintainers of thousands of other packages are working with Tidelift to deliver one enterprise subscription that covers all of the open source you use.

`Learn more <https://tidelift.com/subscription/pkg/pypi-configparser?utm_source=pypi-configparser&utm_medium=referral&utm_campaign=github>`_.

Security Contact
----------------

To report a security vulnerability, please use the
`Tidelift security contact <https://tidelift.com/security>`_.
Tidelift will coordinate the fix and disclosure.

Conversion Process
------------------

This section is technical and should bother you only if you are wondering how
this backport is produced. If the implementation details of this backport are
not important for you, feel free to ignore the following content.

The project takes the following branching approach:

* The ``3.x`` branch holds unchanged files synchronized from the upstream
  CPython repository. The synchronization is currently done by manually copying
  the required files and stating from which CPython changeset they come.

* The ``main`` branch holds a version of the ``3.x`` code with some tweaks
  that make it compatible with older Pythons. Code on this branch must work
  on all supported Python versions. Test with ``tox`` or in CI.

The process works like this:

1. In the ``3.x`` branch, run ``pip-run -- sync-upstream.py``, which
   downloads the latest stable release of Python and copies the relevant
   files from there into their new locations and then commits those
   changes with a nice reference to the relevant upstream commit hash.

2. Check for new names in ``__all__`` and update imports in
   ``configparser.py`` accordingly. Commit.

3. Merge the new commit to ``main``. Run tests. Commit.

4. Make any compatibility changes on ``main``. Run tests. Commit.

5. Update the docs and release the new version.


Footnotes
---------

.. [1] To somewhat ease migration, passing bytestrings is still supported but
       they are converted to Unicode for internal storage anyway. This means
       that for the vast majority of strings used in configuration files, it
       won't matter if you pass them as bytestrings or Unicode. However, if you
       pass a bytestring that cannot be converted to Unicode using the naive
       ASCII codec, a ``UnicodeDecodeError`` will be raised. This is purposeful
       and helps you manage proper encoding for all content you store in
       memory, read from various sources and write back.

.. [2] Life gets much easier when you understand that you basically manage
       **text** in your application.  You don't care about bytes but about
       letters.  In that regard the concept of content encoding is meaningless.
       The only time when you deal with raw bytes is when you write the data to
       a file.  Then you have to specify how your text should be encoded.  On
       the other end, to get meaningful text from a file, the application
       reading it has to know which encoding was used during its creation.  But
       once the bytes are read and properly decoded, all you have is text.  This
       is especially powerful when you start interacting with multiple data
       sources.  Even if each of them uses a different encoding, inside your
       application data is held in abstract text form.  You can program your
       business logic without worrying about which data came from which source.
       You can freely exchange the data you store between sources.  Only
       reading/writing files requires encoding your text to bytes.
