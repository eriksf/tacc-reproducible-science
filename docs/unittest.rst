Unit Testing
============

As our code continues to grow, how can we be sure it is working as expected? If
we make minor changes to the code, what tests can we run to make sure we didn't
break anything? Are our functions written well enough to capture and correctly
handle all of the edge cases we throw at them? In this module, we will use the
Python ``pytest`` library to write unit tests: small tests that are designed to
test specific individual components of code. After working through this module,
students should be able to:

* Find the documentation for the Python ``pytest`` library
* Identify parts of code that should be tested and appropriate assert methods
* Write and run reasonable unit tests


Getting Started
---------------

Unit tests are designed to test small components (e.g. individual functions) of
your code. They should demonstrate that things that are expected to work
actually do work, and things that are expected to break raise appropriate errors.
The Python ``pytest`` unit testing framework supports test automation, set up
and shut down code for tests, and aggregation of tests into collections. It is
not part of the Python Standard Library, so we must install it.

.. code-block:: console

   $ pip install pytest

Find the `documentation here <https://docs.pytest.org/en/7.4.x/>`_.

Pull a copy of the `calculate-pi (https://github.com/eriksf/calculate-pi) <https://github.com/eriksf/calculate-pi>`_
project from GitHub. This is the same code from the containers module but built using
`Poetry <https://python-poetry.org/>`_ and adding in the `Click <https://click.palletsprojects.com/en/8.1.x/>`_
module for creating a command line interface. Poetry is a tool for python packaging and dependency management.
It is a bit like having a dependency manager and a virtual environment rolled into one with the ability
to handle publishing to `PyPI <https://pypi.org/>`_ as well.

To work with an existing poetry project, first install poetry (this project requires python 3.9, so if following
along on Frontera, first ``module load python3/3.9.2``):

.. code-block:: console

   $ curl -sSL https://install.python-poetry.org | python3 -
   $ export PATH=$HOME/.local/bin:$PATH

then grab the project and install

.. code-block:: console

   $ git clone git@github.com:eriksf/calculate-pi.git
   $ cd calculate-pi
   $ poetry install
   Creating virtualenv calculate-pi-mwikXW1c-py3.9 in /home1/03762/eriksf/.cache/pypoetry/virtualenvs
   Installing dependencies from lock file

   Package operations: 13 installs, 0 updates, 0 removals

     • Installing exceptiongroup (1.1.2)
     • Installing iniconfig (2.0.0)
     • Installing packaging (23.1)
     • Installing pluggy (1.2.0)
     • Installing tomli (2.0.1)
     • Installing coverage (7.2.7)
     • Installing mccabe (0.7.0)
     • Installing pycodestyle (2.10.0)
     • Installing pyflakes (3.0.1)
     • Installing pytest (7.4.0)
     • Installing click (8.1.3)
     • Installing flake8 (6.0.0)
     • Installing pytest-cov (4.1.0)

   Installing the current project: calculate-pi (0.1.0)
   $ poetry shell

Let's take a look at the file layout:

.. code-block:: console

   $ tree .
   .
   ├── calculate_pi
   │   ├── __init__.py
   │   └── pi.py
   ├── Dockerfile
   ├── poetry.lock
   ├── pyproject.toml
   ├── README.md
   └── tests
       ├── __init__.py
       ├── responses
       │   └── help.txt
       └── test_calculate_pi.py

   3 directories, 9 files

The important file that controls the package and dependencies is ``pyproject.toml``.

.. code-block:: console

   $ cat pyproject.toml
   [tool.poetry]
   name = "calculate-pi"
   version = "0.1.0"
   description = ""
   authors = ["Erik Ferlanti <eferlanti@tacc.utexas.edu>"]
   readme = "README.md"
   packages = [{include = "calculate_pi"}]

   [tool.poetry.dependencies]
   python = "^3.9"
   click = "^8.1.3"

   [tool.poetry.scripts]
   calculate-pi = "calculate_pi.pi:main"

   [tool.poetry.group.dev.dependencies]
   flake8 = "^6.0.0"
   pytest = "^7.4.0"
   pytest-cov = "^4.1.0"

   [tool.pytest.ini_options]
   addopts = "--verbose"

   [build-system]
   requires = ["poetry-core"]
   build-backend = "poetry.core.masonry.api"

Devise some Reasonable Tests
----------------------------

The functions in this Python3 script are relatively simple, but how can we be
sure they are working as intended? Let's begin with the taking a look at the main
script.

.. code-block:: python3
   :linenos:

   #!/usr/bin/env python3
   import click
   from random import random as r
   from math import pow as p
   from sys import argv

   VERSION = '0.1.0'

   @click.command()
   @click.version_option(VERSION)
   @click.argument('number', type=click.INT, required=True)
   def main(number):
       """Calculate pi using Monte Carlo estimation.

       NUMBER is the number of random points.
       """
       attempts = number
       inside = 0
       tries = 0

       # Try the specified number of random points
       while (tries < attempts):
           tries += 1
           if (p(r(),2) + p(r(),2) < 1):
               inside += 1

       # Compute and print a final ratio
       print( f'Final pi estimate from {attempts} attempts = {4*(inside/tries)}' )

   if __name__ == '__main__':
       main()


In order to speed things up, we have already written a couple of tests and created a test
directory and test script, ``tests/test_calculate_pi.py``. When writing test scripts,
it is a common convention to name them the same name as the script you are testing, but with
the ``test_`` prefix added at the beginning. Let's take a look at the test script:


.. code-block:: python3
   :linenos:

   import os
   import pytest
   from click.testing import CliRunner

   from calculate_pi import pi

   RESPONSE_DIR = os.path.join(os.path.dirname(os.path.abspath(__file__)), 'responses')
   VERSION = '0.1.0'


   def get_response_text(response_file):
       with open(response_file) as f: response_content = f.read()
       return response_content


   class TestCalculatePI(object):

       @pytest.fixture()
       def runner(self):
           return CliRunner()

       def test_print_help_succeeds(self, runner):
           response_text = get_response_text(os.path.join(RESPONSE_DIR, 'help.txt'))
           result = runner.invoke(pi.main, ['--help'])
           assert result.exit_code == 0
           assert result.output == response_text

       def test_print_version_succeeds(self, runner):
           version_string = 'version {}'.format(pi.VERSION)
           result = runner.invoke(pi.main, ['--version'])
           assert result.exit_code == 0
           assert version_string in result.output


Automate Testing with Pytest
----------------------------

Pytest is an excellent framework for small unit tests and for large functional
tests. Because pytest was a development dependency of this project, it should have
been installed when we ran ``poetry install`` above, but let's double check that the
installation worked and there is an executable called ``pytest`` in your PATH:

.. code-block:: console

   $ pytest --version
   pytest 7.4.0


Pytest will automatically look in our working tree for files that start with the
``test_`` prefix, and execute the tests within. Call the ``pytest`` executable in
your top directory, it will find your test function in your test script, run that
function, and finally print some informative output:

.. code-block:: console

   $ pytest
   ========================================================= test session starts =========================================================
   platform linux -- Python 3.9.2, pytest-7.4.0, pluggy-1.2.0 -- /home1/03762/eriksf/.cache/pypoetry/virtualenvs/calculate-pi-mwikXW1c-py3.9/bin/python
   cachedir: .pytest_cache
   rootdir: /home1/03762/eriksf/calculate-pi
   configfile: pyproject.toml
   plugins: cov-4.1.0
   collected 2 items

   tests/test_calculate_pi.py::TestCalculatePI::test_print_help_succeeds PASSED                                                    [ 50%]
   tests/test_calculate_pi.py::TestCalculatePI::test_print_version_succeeds PASSED                                                 [100%]

   ========================================================== 2 passed in 0.26s ==========================================================


What Else Should We Test?
-------------------------

The simple tests we wrote above seem almost trivial, but they are actually great
sanity tests to tell us that our code is working. What other behaviors of our
script should we test? Since this is such a simple script, really the only thing left
to test is the ``main`` function itself.

To test that, let's add the following function to our test script at ``tests/test_calculate_pi.py``:

.. code-block:: python3
   :linenos:
   :emphasize-lines: 34-38

   import os
   import pytest
   from click.testing import CliRunner

   from calculate_pi import pi

   RESPONSE_DIR = os.path.join(os.path.dirname(os.path.abspath(__file__)), 'responses')
   VERSION = '0.1.0'


   def get_response_text(response_file):
       with open(response_file) as f: response_content = f.read()
       return response_content


   class TestCalculatePI(object):

       @pytest.fixture()
       def runner(self):
           return CliRunner()

       def test_print_help_succeeds(self, runner):
           response_text = get_response_text(os.path.join(RESPONSE_DIR, 'help.txt'))
           result = runner.invoke(pi.main, ['--help'])
           assert result.exit_code == 0
           assert result.output == response_text

       def test_print_version_succeeds(self, runner):
           version_string = 'version {}'.format(pi.VERSION)
           result = runner.invoke(pi.main, ['--version'])
           assert result.exit_code == 0
           assert version_string in result.output

       def test_print_final_value(self, runner):
           final_pi = 'Final pi estimate from'
           result = runner.invoke(pi.main, ['10'])
           assert result.exit_code == 0
           assert final_pi in result.output


After adding the above test, run ``pytest`` again:

.. code-block:: console

   $ pytest
   ========================================================= test session starts =========================================================
   platform linux -- Python 3.9.2, pytest-7.4.0, pluggy-1.2.0 -- /home1/03762/eriksf/.cache/pypoetry/virtualenvs/calculate-pi-mwikXW1c-py3.9/bin/python
   cachedir: .pytest_cache
   rootdir: /home1/03762/eriksf/calculate-pi
   configfile: pyproject.toml
   plugins: cov-4.1.0
   collected 3 items

   tests/test_calculate_pi.py::TestCalculatePI::test_print_help_succeeds PASSED                                                    [ 33%]
   tests/test_calculate_pi.py::TestCalculatePI::test_print_version_succeeds PASSED                                                 [ 66%]
   tests/test_calculate_pi.py::TestCalculatePI::test_print_final_value PASSED                                                      [100%]

   ========================================================== 3 passed in 0.06s ==========================================================


Success! The tests for our program are passing. And, if ever we change the code in that program,
we can see if the behavior we intend still passes the test.


Additional Resources
--------------------

* `Pytest documentation <https://docs.pytest.org/en/7.4.x/>`_.
* `Exceptions in Python <https://docs.python.org/3.6/library/exceptions.html>`_
