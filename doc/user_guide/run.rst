================
 Running Pylint
================

From the command line
---------------------

Pylint is meant to be called from the command line. The usage is ::

   pylint [options] modules_or_packages

You should give Pylint the name of a python package or module, or some number
of packages or modules. Pylint
``will not import`` this package or module, though uses Python internals
to locate them and as such is subject to the same rules and configuration.
You should pay attention to your ``PYTHONPATH``, since it is a common error
to analyze an installed version of a module instead of the
development version.

It is also possible to analyze Python files, with a few
restrictions. The thing to keep in mind is that Pylint will try to
convert the file name to a module name, and only be able to process
the file if it succeeds.  ::

  pylint mymodule.py

should always work since the current working
directory is automatically added on top of the python path ::

  pylint directory/mymodule.py

will work if ``directory`` is a python package (i.e. has an __init__.py
file or it is an implicit namespace package) or if "directory" is in the
python path.

By default, pylint will exit with an error when one of the arguments is a directory which is not
a python package. In order to run pylint over all modules and packages within the provided
subtree of a directory, the ``--recursive=y`` option must be provided.

For more details on this see the :ref:`faq`.

From another python program
---------------------------

It is also possible to call Pylint from another Python program,
thanks to the ``Run()`` function in the ``pylint.lint`` module
(assuming Pylint options are stored in a list of strings ``pylint_options``) as:

.. sourcecode:: python

  import pylint.lint
  pylint_opts = ['--disable=line-too-long', 'myfile.py']
  pylint.lint.Run(pylint_opts)

Another option would be to use the ``run_pylint`` function, which is the same function
called by the command line. You can either patch ``sys.argv`` or supply arguments yourself:

.. sourcecode:: python

  import pylint

  sys.argv = ["pylint", "your_file"]
  pylint.run_pylint()
  # Or:
  pylint.run_pylint(argv=["your_file"])

To silently run Pylint on a ``module_name.py`` module,
and get its standard output and error:

.. sourcecode:: python

  from pylint import epylint as lint

  (pylint_stdout, pylint_stderr) = lint.py_run('module_name.py', return_std=True)

It is also possible to include additional Pylint options in the first argument to ``py_run``:

.. sourcecode:: python

  from pylint import epylint as lint

  (pylint_stdout, pylint_stderr) = lint.py_run('module_name.py --disable C0114', return_std=True)

The options ``--msg-template="{path}:{line}: {category} ({msg_id}, {symbol}, {obj}) {msg}"`` and
``--reports=n`` are set implicitly inside the ``epylint`` module.

Finally, it is possible to invoke pylint programmatically with a
reporter initialized with a custom stream:

.. sourcecode:: python

    from io import StringIO

    from pylint.lint import Run
    from pylint.reporters.text import TextReporter

    pylint_output = StringIO()  # Custom open stream
    reporter = TextReporter(pylint_output)
    Run(["test_file.py"], reporter=reporter, do_exit=False)
    print(pylint_output.getvalue())  # Retrieve and print the text report

The reporter can accept any stream object as as parameter. In this example,
the stream outputs to a file:

.. sourcecode:: python

    from pylint.lint import Run
    from pylint.reporters.text import TextReporter

    with open("report.out", "w") as f:
        reporter = TextReporter(f)
        Run(["test_file.py"], reporter=reporter, do_exit=False)

This would be useful to capture pylint output in an open stream which
can be passed onto another program.


Command line options
--------------------

First of all, we have two basic (but useful) options.

--version             show program's version number and exit
-h, --help            show help about the command line options

Pylint is architected around several checkers. You can disable a specific
checker or some of its messages or message categories by specifying
``--disable=<symbol>``. If you want to enable only some checkers or some
message symbols, first use ``--disable=all`` then
``--enable=<symbol>`` with ``<symbol>`` being a comma-separated list of checker
names and message symbols. See the list of available features for a
description of provided checkers with their functionalities.
The ``--disable`` and ``--enable`` options can be used with comma-separated lists
mixing checkers, message ids and categories like ``-d C,W,no-error,design``

It is possible to disable all messages with ``--disable=all``. This is
useful to enable only a few checkers or a few messages by first
disabling everything, and then re-enabling only what you need.

Each checker has some specific options, which can take either a yes/no
value, an integer, a python regular expression, or a comma-separated
list of values (which are generally used to override a regular
expression in special cases). For a full list of options, use ``--help``

Specifying all the options suitable for your setup and coding
standards can be tedious, so it is possible to use a configuration file to
specify the default values.  You can specify a configuration file on the
command line using the ``--rcfile`` option.  Otherwise, Pylint searches for a
configuration file in the following order and uses the first one it finds:

#. ``pylintrc`` in the current working directory
#. ``.pylintrc`` in the current working directory
#. ``pyproject.toml`` in the current working directory,
   providing it has at least one ``tool.pylint.`` section.
   The ``pyproject.toml`` must prepend section names with ``tool.pylint.``,
   for example ``[tool.pylint.'MESSAGES CONTROL']``. They can also be passed
   in on the command line.
#. ``setup.cfg`` in the current working directory,
   providing it has at least one ``pylint.`` section
#. If the current working directory is in a Python package, Pylint searches \
   up the hierarchy of Python packages until it finds a ``pylintrc`` file. \
   This allows you to specify coding standards on a module-by-module \
   basis.  Of course, a directory is judged to be a Python package if it \
   contains an ``__init__.py`` file.
#. The file named by environment variable ``PYLINTRC``
#. if you have a home directory which isn't ``/root``:

   #. ``.pylintrc`` in your home directory
   #. ``.config/pylintrc`` in your home directory

#. ``/etc/pylintrc``

The ``--generate-rcfile`` option will generate a commented configuration file
on standard output according to the current configuration and exit. This
includes:

* Any configuration file found as explained above
* Options appearing before ``--generate-rcfile`` on the Pylint command line

Of course you can also start with the default values and hand-tune the
configuration.

Other useful global options include:

--ignore=<file[,file...]>  Files or directories to be skipped. They should be
                           base names, not paths.
--output-format=<format>   Select output format (text, json, custom).
--msg-template=<template>  Modify text output message template.
--list-msgs                Generate pylint's messages.
--list-msgs-enabled        Display a list of what messages are enabled and
                           disabled with the given configuration.
--full-documentation       Generate pylint's full documentation, in reST
                             format.

Parallel execution
------------------

It is possible to speed up the execution of Pylint. If the running computer
has more CPUs than one, then the work for checking all files could be spread across all
cores via Pylints's sub-processes.
This functionality is exposed via the ``-j`` command-line parameter.
If the provided number is 0, then the total number of CPUs will be autodetected and used.

Example::

  pylint -j 4 mymodule1.py mymodule2.py mymodule3.py mymodule4.py

This will spawn 4 parallel Pylint sub-process, where each provided module will
be checked in parallel. Discovered problems by checkers are not displayed
immediately. They are shown just after checking a module is complete.

There are some limitations in running checks in parallel in the current
implementation. It is not possible to use custom plugins
(i.e. ``--load-plugins`` option), nor it is not possible to use
initialization hooks (i.e. the ``--init-hook`` option).

Exit codes
----------

Pylint returns bit-encoded exit codes.

=========  =========================
exit code  meaning
=========  =========================
0          no error
1          fatal message issued
2          error message issued
4          warning message issued
8          refactor message issued
16         convention message issued
32         usage error
=========  =========================

For example, an exit code of ``20`` means there was at least one warning message (4)
and at least one convention message (16) and nothing else.
