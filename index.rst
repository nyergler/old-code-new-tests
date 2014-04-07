.. role:: strike
    :class: strike

=====================
 Old Code, New Tests
=====================

.. ifslides::

   .. figure:: /_static/old-code.jpg
      :class: fill

      CC BY-NC-SA https://www.flickr.com/photos/lexpaul/10199344214/

Background
==========

Historically
------------

.. rst-class:: build

* Eventbrite started in 2007
* No testing culture to speak of
* Started writing some tests when we adopted Django
* Unreasonably high regression level in 2011

Today
-----

.. rst-class:: build

* Bigger team
* More complex architecture
* Robust test suite
* Fewer regressions
* "Tests matter"

What do we test?
================

Unit Tests
----------

* Tests discreet methods, functions
* "Django-style": the unit includes the database
* Database ops in one test isolated from others
* Ideally fast enough you can do reasonable TDD

Integration Tests
-----------------

* Test integration between our services
* Look almost identical to unit tests
* Database ops in one test isolated from others
* Typically slower that unit tests
* Run automatically as part of our build process

Functional Tests
----------------

* Selenium testing via Sauce Labs
* Tests end-to-end functionality of the site
* Coverage for critical features
* Database reset between test *runs*

Testing Existing Code Bases
===========================

Issues
------

* Tests guide how you write code
* Untested, code often grows "organically"
* Understanding intended behavior is difficult

Work backwards
--------------

* Start with functional tests
* Easiest to write for existing code
* Canary in the coal mine
* Lets you know when something breaks

Stop digging
------------

* Set expectations about new code
* New code *must* have tests
* Good practice for testing and composing code
* Necessary skill before you can tackle refactorings

Small Wins Count
----------------

* Old code needs refactoring to unit test
* Not every refactoring need be huge

What Gets in the Way?
=====================

Environmental Impact
--------------------

* Ship what you test
* Dependencies, settings, etc all can impact tests

Jenkins
-------

.. figure:: /_static/jenkins.jpg

.. nextslide::

* Main build includes all dependencies
* Single artifact for testing and shipping
* Ensures everyone knows about failures
* Also helps mitigate test speed issues

Settings Guard
--------------

* DB isolation isn't enough
* Guards against changing settings
* Changing settings make test failures appear mysterious
* Plugin fails tests if important settings change

Settings Guard
--------------

.. code-block:: python

   from nose.plugins import Plugin

   class SettingsGuard(Plugin):
       """Protect Settings against mutation during tests."""

       name = 'settingsguard'

       def configure(self, options, conf):
           super(SettingsGuard, self).configure(options, conf)

           # store the initial state of settings
           ...

       def stopTest(self, test):

           # confirm that the settings haven't changed
           test.assertEqual(self._is_prod, self._core_globals.gIsProd,
                            "gIsProd mutated.")


Polyglot Code
-------------

* Python + Javascript for Eventbrite
* Wanted to start testing new Javascript code
* Single command for engineers to use
* Wrote Python TestCase_ subclass to include JS tests

JasmineTestCase
----------------

.. code-block:: python

   class JasmineTests(TestCase):

       ...

       @parameterized.expand(get_suite_names)
       def test_jasmine_suite(self, spec_suite):

           spec_suite_config = get_javascript_spec_suites().get(
               spec_suite,
               None,
           )

           # call the testrunner view and write the contents to disk
           # then execute with PhantomJS ...

           for line in results.stdout:
               system_error = line.rfind('FAILED:') > -1
               if system_error:
                   self.fail_with_description(description, system_error)

Need for Speed
--------------

* We wrote functional tests first
* Often perform poorly
* Needed to run the "right" tests

Nose Knows
----------

https://github.com/eventbrite/nose-knows

* Only run tests that touch changed code path
* Run once to record mappings
* Give it a file and it runs tests that touch that file

.. nextslide::

.. code-block:: bash
   :emphasize-lines: 3

   $ nosetests --with-knows \
     --knows-dir=$BASE_DIR \
     load_data/sql_utils/checkpoints.py
   .......
   ----------------------------------------------------------------------
   Ran 7 tests in 0.003s

   OK

Old Code, New Tests
-------------------

* Never too late to start testing
* Add tests in reverse
* Figure out what's stopping engineers from testing

Thanks!
-------

* http://github.com/nyergler/old-code-new-tests
* nathan@eventbrite.com
* `@nyergler`_

.. _TestCase: https://docs.python.org/2/library/unittest.html#unittest.TestCase
.. _`@nyergler`: http://twitter.com/nyergler
.. _`Settings Guard`:
