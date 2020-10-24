FORKED FROM https://github.com/pyutils/line_profiler

IMPORTANT NOTES:

In kernprofile.py, it overwrites execfile() which reads the .py script to run and compiles it. the new function injects code that wraps all functions and classes (or ones that are included/excluded manually) in the script with @Profile before it runs execfile. This might be considered a security risk since there is code being injected and it takes input from the command line but since you are the one in control running a python script idk if thats something to be concerned about. It also breaks compatibility with python 2. but since python 2 is depreciated idk if its worth the time to make it work.

In line_profiler.py, it changes abit of the inner logic of identifying and profiling functions and classes that are wrapped with the @Profile decorator which could potentially break it for your usage as I have only tested this with windows 10.



CHANGES TO THIS REPO:

1. command-line option to profile all functions:

kernprof with line_profiler requires the user to manually decorate the functions they want to profile as it is not feasible to profile all the code.
This can be arduous to decorate all functions, and in some cases it might not be possible to modify the script that they want to profile. And the @Profile decorator which is a builtin can throw errors when trying to run the code normally without adding 
    
    import line_profiler
    
    profile = line_profiler.LineProfiler()

or

    if type(__builtins__) is not dict or 'profile' not in __builtins__: profile=lambda f:f

These command line arguments will allow the user to decorate all functions and classes inside the specified script (shallow, only profiles the current script) from the command line. which solves the problem of manually decorating all the functions, or if the user cannot modify the script.
It does this by injecting code that adds the @Profile decorator to all the functions and classes in the code that is read from the py file before exec is called

    argument '-a', '--auto' to automatically profile all functions and classes in script
    
    argument '-i', '--include' to specify which functions/classes to profile (comma separated)
    
    argument '-x', '--exclude' to specify which functions/classes to not profile (comma separated)
    
    argument '-m', '--module' to pass a single module's name in the script to profile (can also take function or classes, similar to include, but include does not work with modules whereas this argument passes the name as a variable directly in the code (which can cause it to fail if the name is undefined) as opposed to list of comma separated string in include)
    
    argument '-f', '--auto-find' find text in script to insert profiling code before/after (the profiling code needs to be added after all function, class and module declarations but before they are run). defaults to "if __name__ == '__main__':"
    
    argument '-r', '--auto-before' specify whether profiling code must be added before or after "--auto-find". defaults to after

example usage:

1) profile all functions and classes in script.py:

    kernprof.exe -o ./script.lprof -b -l -a script.py

    python -m line_profiler ./script.lprof -k -u 1e-6 > ./script.lprofout


2) profile only these functions and classes "main" and "foo":

    kernprof.exe -b -l -a -i "main,foo" script.py


3) profile all the functions and classes except "bar":

    kernprof.exe -b -l -a -x "bar" script.py


4) profile the module "module1" imported inside script:

    kernprof.exe -b -l -a -m "module1" script.py


5) insert profiling code before foo function execution:

    kernprof.exe -b -l -a -r -f "foo()" script.py


2. command-line option to skip profiling certain lines:

add a command line option -x --ignore that takes a comma separated string to remove the timings of lines that contain any of the text in a the string
Since the text is comma separated, you cannot filter using text that has a comma in it. percent represents the percentage of filtered time (non-ignored lines) / real time


3. command-line option to ignore non-run functions:

add command line option -k, --skip to skip displaying functions that have no runtime


4. command-line option to control unit of time:

add a command line option -u --unit to specify the unit of time displayed by line_profiler for ease of use and ease of reading the result.
Windows defaults to 1e-7 whereas specifying 1e-6 would be easier to read and mentally parse.
The user can easily change the unit if the runtimes are expected to be long eg: seconds

eg: to specify a time unit of 1us on the output of kernprof "script.lprof"

    python -m line_profiler ./script.lprof -u 1e-6




Putting it all together:

I use sublime text's commands to automatically run kernprof.exe on the currently open file with all my desired command line options. I specify where i want the kernprof output to be saved which will then need to be converted into readable text by line_profiler.

    cmd /k cd /d "%PATH_TO_CURRENT_SCRIPT_FOLDER%" && "%PATH_TO_PYTHON_ENV%/Scripts/kernprof.exe" -o "%PATH_TO_STORE_PROFILER_OUTPUT%/script.lprof" -b -l -a "%PATH_TO_CURRENT_SCRIPT_FILE%"

Then to convert the kernprof output to readable text using line_profiler. I specify to ignore lines that containt the words import, print( and a few opencv functions. I save the line_profiler output as .py because it applies sublime text's python syntax highlighting

    cmd /k "%PATH_TO_PYTHON_ENV%/python.exe" -m line_profiler "%PATH_TO_STORE_PROFILER_OUTPUT%/script.lprof" -k -u 1e-6 -x "import,print(,cv2.waitKey(,cv2.imshow(,cv2.destroy,cv2.namedWindow(" > "%PATH_TO_STORE_PROFILER_OUTPUT%/script.lprof.py'

You can then open the output file to view the profiler output







line_profiler and kernprof
--------------------------

|Pypi| |Downloads| |Travis|


NOTICE: This is the official `line_profiler` repository. The most recent
version of `line-profiler <https://pypi.org/project/line_profiler/>`_ on pypi
points to this repo. The the original 
`line_profiler <https://github.com/rkern/line_profiler/>`_ package by  
`@rkern <https://github.com/rkern/>`_ is currently unmaintained. This fork
seeks to simply maintain the original code so it continues to work in new
versions of Python.

----


`line_profiler` is a module for doing line-by-line profiling of functions.
kernprof is a convenient script for running either `line_profiler` or the Python
standard library's cProfile or profile modules, depending on what is available.

They are available under a `BSD license`_.

.. _BSD license: https://raw.githubusercontent.com/pyutils/line_profiler/master/LICENSE.txt

.. contents::


Installation
============

Releases of `line_profiler` can be installed using pip::

    $ pip install line_profiler

Source releases and any binaries can be downloaded from the PyPI link.

    http://pypi.python.org/pypi/line_profiler

To check out the development sources, you can use Git_::

    $ git clone https://github.com/pyutils/line_profiler.git

You may also download source tarballs of any snapshot from that URL.

Source releases will require a C compiler in order to build `line_profiler`.
In addition, git checkouts will also require Cython_ >= 0.10. Source releases
on PyPI should contain the pregenerated C sources, so Cython should not be
required in that case.

`kernprof` is a single-file pure Python script and does not require
a compiler.  If you wish to use it to run cProfile and not line-by-line
profiling, you may copy it to a directory on your `PATH` manually and avoid
trying to build any C extensions.

.. _git: http://git-scm.com/
.. _Cython: http://www.cython.org
.. _build and install: http://docs.python.org/install/index.html


line_profiler
=============

The current profiling tools supported in Python 2.7 and later only time
function calls. This is a good first step for locating hotspots in one's program
and is frequently all one needs to do to optimize the program. However,
sometimes the cause of the hotspot is actually a single line in the function,
and that line may not be obvious from just reading the source code. These cases
are particularly frequent in scientific computing. Functions tend to be larger
(sometimes because of legitimate algorithmic complexity, sometimes because the
programmer is still trying to write FORTRAN code), and a single statement
without function calls can trigger lots of computation when using libraries like
numpy. cProfile only times explicit function calls, not special methods called
because of syntax. Consequently, a relatively slow numpy operation on large
arrays like this, ::

    a[large_index_array] = some_other_large_array

is a hotspot that never gets broken out by cProfile because there is no explicit
function call in that statement.

LineProfiler can be given functions to profile, and it will time the execution
of each individual line inside those functions. In a typical workflow, one only
cares about line timings of a few functions because wading through the results
of timing every single line of code would be overwhelming. However, LineProfiler
does need to be explicitly told what functions to profile. The easiest way to
get started is to use the `kernprof` script. ::

    $ kernprof -l script_to_profile.py

`kernprof` will create an instance of LineProfiler and insert it into the
`__builtins__` namespace with the name `profile`. It has been written to be
used as a decorator, so in your script, you decorate the functions you want
to profile with @profile. ::

    @profile
    def slow_function(a, b, c):
        ...

The default behavior of `kernprof` is to put the results into a binary file
script_to_profile.py.lprof . You can tell `kernprof` to immediately view the
formatted results at the terminal with the [-v/--view] option. Otherwise, you
can view the results later like so::

    $ python -m line_profiler script_to_profile.py.lprof

For example, here are the results of profiling a single function from
a decorated version of the pystone.py benchmark (the first two lines are output
from `pystone.py`, not `kernprof`)::

    Pystone(1.1) time for 50000 passes = 2.48
    This machine benchmarks at 20161.3 pystones/second
    Wrote profile results to pystone.py.lprof
    Timer unit: 1e-06 s

    File: pystone.py
    Function: Proc2 at line 149
    Total time: 0.606656 s

    Line #      Hits         Time  Per Hit   % Time  Line Contents
    ==============================================================
       149                                           @profile
       150                                           def Proc2(IntParIO):
       151     50000        82003      1.6     13.5      IntLoc = IntParIO + 10
       152     50000        63162      1.3     10.4      while 1:
       153     50000        69065      1.4     11.4          if Char1Glob == 'A':
       154     50000        66354      1.3     10.9              IntLoc = IntLoc - 1
       155     50000        67263      1.3     11.1              IntParIO = IntLoc - IntGlob
       156     50000        65494      1.3     10.8              EnumLoc = Ident1
       157     50000        68001      1.4     11.2          if EnumLoc == Ident1:
       158     50000        63739      1.3     10.5              break
       159     50000        61575      1.2     10.1      return IntParIO


The source code of the function is printed with the timing information for each
line. There are six columns of information.

    * Line #: The line number in the file.

    * Hits: The number of times that line was executed.

    * Time: The total amount of time spent executing the line in the timer's
      units. In the header information before the tables, you will see a line
      "Timer unit:" giving the conversion factor to seconds. It may be different
      on different systems.

    * Per Hit: The average amount of time spent executing the line once in the
      timer's units.

    * % Time: The percentage of time spent on that line relative to the total
      amount of recorded time spent in the function.

    * Line Contents: The actual source code. Note that this is always read from
      disk when the formatted results are viewed, *not* when the code was
      executed. If you have edited the file in the meantime, the lines will not
      match up, and the formatter may not even be able to locate the function
      for display.

If you are using IPython, there is an implementation of an %lprun magic command
which will let you specify functions to profile and a statement to execute. It
will also add its LineProfiler instance into the __builtins__, but typically,
you would not use it like that.

For IPython 0.11+, you can install it by editing the IPython configuration file
`~/.ipython/profile_default/ipython_config.py` to add the `'line_profiler'`
item to the extensions list::

    c.TerminalIPythonApp.extensions = [
        'line_profiler',
    ]


To get usage help for %lprun, use the standard IPython help mechanism::

    In [1]: %lprun?

These two methods are expected to be the most frequent user-level ways of using
LineProfiler and will usually be the easiest. However, if you are building other
tools with LineProfiler, you will need to use the API. There are two ways to
inform LineProfiler of functions to profile: you can pass them as arguments to
the constructor or use the `add_function(f)` method after instantiation. ::

    profile = LineProfiler(f, g)
    profile.add_function(h)

LineProfiler has the same `run()`, `runctx()`, and `runcall()` methods as
cProfile.Profile as well as `enable()` and `disable()`. It should be noted,
though, that `enable()` and `disable()` are not entirely safe when nested.
Nesting is common when using LineProfiler as a decorator. In order to support
nesting, use `enable_by_count()` and `disable_by_count()`. These functions will
increment and decrement a counter and only actually enable or disable the
profiler when the count transitions from or to 0.

After profiling, the `dump_stats(filename)` method will pickle the results out
to the given file. `print_stats([stream])` will print the formatted results to
sys.stdout or whatever stream you specify. `get_stats()` will return LineStats
object, which just holds two attributes: a dictionary containing the results and
the timer unit.


kernprof
========

`kernprof` also works with cProfile, its third-party incarnation lsprof, or the
pure-Python profile module depending on what is available. It has a few main
features:

    * Encapsulation of profiling concerns. You do not have to modify your script
      in order to initiate profiling and save the results. Unless if you want to
      use the advanced __builtins__ features, of course.

    * Robust script execution. Many scripts require things like __name__,
      __file__, and sys.path to be set relative to it. A naive approach at
      encapsulation would just use execfile(), but many scripts which rely on
      that information will fail. kernprof will set those variables correctly
      before executing the script.

    * Easy executable location. If you are profiling an application installed on
      your PATH, you can just give the name of the executable. If kernprof does
      not find the given script in the current directory, it will search your
      PATH for it.

    * Inserting the profiler into __builtins__. Sometimes, you just want to
      profile a small part of your code. With the [-b/--builtin] argument, the
      Profiler will be instantiated and inserted into your __builtins__ with the
      name "profile". Like LineProfiler, it may be used as a decorator, or
      enabled/disabled with `enable_by_count()` and `disable_by_count()`, or
      even as a context manager with the "with profile:" statement.

    * Pre-profiling setup. With the [-s/--setup] option, you can provide
      a script which will be executed without profiling before executing the
      main script. This is typically useful for cases where imports of large
      libraries like wxPython or VTK are interfering with your results. If you
      can modify your source code, the __builtins__ approach may be
      easier.

The results of profile script_to_profile.py will be written to
script_to_profile.py.prof by default. It will be a typical marshalled file that
can be read with pstats.Stats(). They may be interactively viewed with the
command::

    $ python -m pstats script_to_profile.py.prof

Such files may also be viewed with graphical tools like kcachegrind_ through the
converter program pyprof2calltree_ or RunSnakeRun_.

.. _kcachegrind: http://kcachegrind.sourceforge.net/html/Home.html
.. _pyprof2calltree: http://pypi.python.org/pypi/pyprof2calltree/
.. _RunSnakeRun: http://www.vrplumber.com/programming/runsnakerun/


Frequently Asked Questions
==========================

* Why the name "kernprof"?

    I didn't manage to come up with a meaningful name, so I named it after
    myself.

* Why not use hotshot instead of line_profile?

    hotshot can do line-by-line timings, too. However, it is deprecated and may
    disappear from the standard library. Also, it can take a long time to
    process the results while I want quick turnaround in my workflows. hotshot
    pays this processing time in order to make itself minimally intrusive to the
    code it is profiling. Code that does network operations, for example, may
    even go down different code paths if profiling slows down execution too
    much. For my use cases, and I think those of many other people, their
    line-by-line profiling is not affected much by this concern.

* Why not allow using hotshot from kernprof.py?

    I don't use hotshot, myself. I will accept contributions in this vein,
    though.

* The line-by-line timings don't add up when one profiled function calls
  another. What's up with that?

    Let's say you have function F() calling function G(), and you are using
    LineProfiler on both. The total time reported for G() is less than the time
    reported on the line in F() that calls G(). The reason is that I'm being
    reasonably clever (and possibly too clever) in recording the times.
    Basically, I try to prevent recording the time spent inside LineProfiler
    doing all of the bookkeeping for each line. Each time Python's tracing
    facility issues a line event (which happens just before a line actually gets
    executed), LineProfiler will find two timestamps, one at the beginning
    before it does anything (t_begin) and one as close to the end as possible
    (t_end). Almost all of the overhead of LineProfiler's data structures
    happens in between these two times.

    When a line event comes in, LineProfiler finds the function it belongs to.
    If it's the first line in the function, we record the line number and
    *t_end* associated with the function. The next time we see a line event
    belonging to that function, we take t_begin of the new event and subtract
    the old t_end from it to find the amount of time spent in the old line. Then
    we record the new t_end as the active line for this function. This way, we
    are removing most of LineProfiler's overhead from the results. Well almost.
    When one profiled function F calls another profiled function G, the line in
    F that calls G basically records the total time spent executing the line,
    which includes the time spent inside the profiler while inside G.

    The first time this question was asked, the questioner had the G() function
    call as part of a larger expression, and he wanted to try to estimate how
    much time was being spent in the function as opposed to the rest of the
    expression. My response was that, even if I could remove the effect, it
    might still be misleading. G() might be called elsewhere, not just from the
    relevant line in F(). The workaround would be to modify the code to split it
    up into two lines, one which just assigns the result of G() to a temporary
    variable and the other with the rest of the expression.

    I am open to suggestions on how to make this more robust. Or simple
    admonitions against trying to be clever.

* Why do my list comprehensions have so many hits when I use the LineProfiler?

    LineProfiler records the line with the list comprehension once for each
    iteration of the list comprehension.

* Why is kernprof distributed with line_profiler? It works with just cProfile,
  right?

    Partly because kernprof.py is essential to using line_profiler effectively,
    but mostly because I'm lazy and don't want to maintain the overhead of two
    projects for modules as small as these. However, kernprof.py is
    a standalone, pure Python script that can be used to do function profiling
    with just the Python standard library. You may grab it and install it by
    itself without `line_profiler`.

* Do I need a C compiler to build `line_profiler`? kernprof.py?

    You do need a C compiler for line_profiler. kernprof.py is a pure Python
    script and can be installed separately, though.

* Do I need Cython to build `line_profiler`?

    You should not have to if you are building from a released source tarball.
    It should contain the generated C sources already. If you are running into
    problems, that may be a bug; let me know. If you are building from
    a git checkout or snapshot, you will need Cython to generate the
    C sources. You will probably need version 0.10 or higher. There is a bug in
    some earlier versions in how it handles NULL PyObject* pointers.

    As of version ``3.0.0`` manylinux wheels containing the binaries are
    available on pypi. Work is still needed to publish osx and win32 wheels.
    (PRs for this would be helpful!)

* What version of Python do I need?

    Both `line_profiler` and `kernprof` have been tested with Python 2.7, and
    3.5-3.8.


To Do
=====

cProfile uses a neat "rotating trees" data structure to minimize the overhead of
looking up and recording entries. LineProfiler uses Python dictionaries and
extension objects thanks to Cython. This mostly started out as a prototype that
I wanted to play with as quickly as possible, so I passed on stealing the
rotating trees for now. As usual, I got it working, and it seems to have
acceptable performance, so I am much less motivated to use a different strategy
now. Maybe later. Contributions accepted!


Bugs and Such
=============

Bugs and pull requested can be submitted on GitHub_.

.. _GitHub: https://github.com/pyutils/line_profiler


Changes
=======

See `CHANGELOG`_.

.. _CHANGELOG: CHANGELOG.rst


.. |CircleCI| image:: https://circleci.com/gh/pyutils/line_profiler.svg?style=svg
    :target: https://circleci.com/gh/pyutils/line_profiler
.. |Travis| image:: https://img.shields.io/travis/pyutils/line_profiler/master.svg?label=Travis%20CI
   :target: https://travis-ci.org/pyutils/line_profiler?branch=master
.. |Appveyor| image:: https://ci.appveyor.com/api/projects/status/github/pyutils/line_profiler?branch=master&svg=True
   :target: https://ci.appveyor.com/project/pyutils/line_profiler/branch/master
.. |Codecov| image:: https://codecov.io/github/pyutils/line_profiler/badge.svg?branch=master&service=github
   :target: https://codecov.io/github/pyutils/line_profiler?branch=master
.. |Pypi| image:: https://img.shields.io/pypi/v/line_profiler.svg
   :target: https://pypi.python.org/pypi/line_profiler
.. |Downloads| image:: https://img.shields.io/pypi/dm/line_profiler.svg
   :target: https://pypistats.org/packages/line_profiler
.. |ReadTheDocs| image:: https://readthedocs.org/projects/line_profiler/badge/?version=latest
    :target: http://line_profiler.readthedocs.io/en/latest/
