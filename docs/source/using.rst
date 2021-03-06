Using airspeed velocity
=======================

**airspeed velocity** is designed to benchmark a single project over
its lifetime using a given set of benchmarks.  Below, we use the
phrase "project" to refer to the project being benchmarked, and
"benchmark suite" to refer to the set of benchmarks -- i.e., little
snippets of code that are timed -- being run against the project.  The
benchmark suite may live inside the project's repository, or it may
reside in a separate repository -- the choice is up to you and is
primarily a matter of style or policy.  Importantly, the result data
stored alongside the benchmark suite may grow quite large, which is a
good reason to not include it in the main project repository.

The user interacts with **airspeed velocity** through the ``asv``
command.  Like ``git``, the ``asv`` command has a number of
"subcommands" for performing various actions on your benchmarking
project.

Setting up a new benchmarking project
-------------------------------------

The first thing to do is to set up an **airspeed velocity** benchmark
suite for your project.  It must contain, at a minimum, a single
configuration file, ``asv.conf.json``, and a directory tree of Python
files containing benchmarks.

The ``asv quickstart`` command can be used to create a new
benchmarking suite.  Change to the directory where you would like your
new benchmarking suite to be created and run::

    $ asv quickstart
    Edit asv.conf.json to get started.

Now that you have the bare bones of a benchmarking suite, let's edit
the configuration file, ``asv.conf.json``.  Like most files that
**airspeed velocity** uses and generates, it is a JSON file.

There are comments in the file describing what each of the elements
do, and there is also a :ref:`conf-reference` with more details.  The
values that will most likely need to be changed for any benchmarking
suite are:

- ``project``: The name of the project being benchmarked.

- ``project_url``: The project's homepage.

- ``repo``: The URL to the DVCS repository for the project.  This
  should be a read-only URL so that anyone, even those without commit
  rights to the repository, can run the benchmarks.  For a project on
  github, for example, the URL would look like:
  ``https://github.com/spacetelescope/asv.git``

- ``show_commit_url``: The base of URLs used to display commits for
  the project.  This allows users to click on a commit in the web
  interface and have it display the contents of that commit.  For a
  github project, the URL is of the form
  ``http://github.com/$OWNER/$REPO/commit/``.

The rest of the values can usually be left to their defaults, unless
you want to benchmark against multiple versions of Python or multiple
versions of third-party dependencies.

Once you've set up the project's configuration, you'll need to write
some benchmarks.  The benchmarks live in Python files in the
``benchmarks`` directory.  The ``quickstart`` command has created a
single example benchmark file already in
``benchmarks/benchmarks.py``::

    class TimeSuite:
        """
        An example benchmark that times the performance of various kinds
        of iterating over dictionaries in Python.
        """
        def setup(self):
            self.d = {}
            for x in range(500):
                self.d[x] = None

        def time_keys(self):
            for key in self.d.keys():
                pass

        def time_iterkeys(self):
            for key in self.d.iterkeys():
                pass

        def time_range(self):
            d = self.d
            for key in range(500):
                x = d[key]

        def time_xrange(self):
            d = self.d
            for key in xrange(500):
                x = d[key]

You'll want to replace these benchmarks with your own.  See
:ref:`writing-benchmarks` for more information.

Running benchmarks
------------------

Benchmarks are run using the ``asv run`` subcommand.

Let's start by just benchmarking the latest commit in the current ``master`` of
the project::

    $ asv run

Machine information
```````````````````

If this is the first time using ``asv run`` on a given machine, (which
it probably is, if you're following along), you will be prompted for
information about the machine, such as its platform, cpu and memory.
**airspeed velocity** will try to make reasonable guesses, so it's
usually ok to just press ``Enter`` to accept each default value.  This
information is stored in the ``.asv-machine.json`` file in your home
directory::


    I will now ask you some questions about this machine to identify
    it in the benchmarks.

    1. machine: A unique name to identify this machine in the results.
       May be anything, as long as it is unique across all the
       machines used to benchmark this project.
    machine [cheetah]:
    2. os: The OS type and version of this machine.  For example,
       'Macintosh OS-X 10.8'.
    os [Linux 3.12.7-300.fc20.x86_64]:
    3. arch: The generic CPU architecture of this machine.  For
       example, 'i386' or 'x86_64'.
    arch [x86_64]:
    4. cpu: A specific description of the CPU of this machine,
       including its speed and class.  For example, 'Intel(R) Core(TM)
       i5-2520M CPU @ 2.50GHz (4 cores)'.
    cpu [Intel(R) Core(TM) i5-2520M CPU @ 2.50GHz (4 cores)]:
    5. ram: The amount of physical RAM on this machine.  For example,
       '4GB'.
    ram []:

.. note::

    If you ever need to update the machine information later, you can
    run ``asv machine``.

Environments
````````````

Next, the Python virtual environments will be set up: one for each of
the combinations of Python versions and the matrix of project
dependencies, if any.  The first time this is run, this may take some
time, as many files are copied over and dependencies are installed
into the environment.  The environments are stored in the ``env``
directory so that the next time the benchmarks are run, things will
start much faster.

.. note::

    ``asv`` does not build Pythons for you, but it expects to find
    each of the Python versions specified in the ``asv.conf.json``
    file available on the ``PATH``.  For example, if the
    ``asv.conf.json`` file has::

        "pythons": ["2.7", "3.3"]

    then it will use the executables named ``python2.7`` and
    ``python3.3`` on the path.  There are many ways to get multiple
    versions of Python installed -- your package manager, ``apt-get``,
    ``yum``, ``MacPorts`` or ``homebrew`` probably has them, or you
    can also use `pyenv <https://github.com/yyuu/pyenv>`__.  ``asv``
    always works in a virtual environment, so it will not change what
    is installed in any of the python environments on your system.

Benchmarking
````````````

Finally, the benchmarks are run::

    $ asv run
              - Cloning project.
              - Fetching recent changes
              - Creating virtualenvs
              -- Creating virtualenv for py2.7.
              - Installing dependencies
              -- Upgrading setuptools in py2.7.
              - Discovering benchmarks
              -- Fetching recent changes
              -- Creating virtualenv for py2.7
              -- Upgrading setuptools in py2.7.
              -- Uninstalling asv from py2.7
              -- Installing /home/mdboom/Work/builds/asv.tmp/asv into py2.7.
              - Running 5 total benchmarks (1 commits * 1
                environments * 5 benchmarks)
    [  0.00%] - For asv commit hash 14ce7984:
    [  0.00%] -- Building for py2.7
    [  0.00%] --- Uninstalling asv from py2.7
    [  0.00%] --- Installing /home/mdboom/Work/builds/asv.tmp/asv into py2.7
    [  0.00%] -- Benchmarking py2.7
    [ 20.00%] --- Running benchmarks.TimeSuite.time_xrange          30.23μs
    [ 40.00%] --- Running benchmarks.TimeSuite.time_range           31.85μs
    [ 60.00%] --- Running benchmarks.MemSuite.mem_list                 2.4k
    [ 80.00%] --- Running benchmarks.TimeSuite.time_iterkeys         9.86μs
    [100.00%] --- Running benchmarks.TimeSuite.time_keys            10.29μs

To improve reproducibility, each benchmark is run in its own process.

The killer feature of **airspeed velocity** is that it can track the
benchmark performance of your project over time.  The required
``range`` argument to ``asv run`` specifies a range of commits that
should be benchmarked.  The value of this argument is passed directly
to ``git log`` to get the set of commits, so it actually has a very
powerful syntax defined in the `gitrevisions manpage
<https://www.kernel.org/pub/software/scm/git/docs/gitrevisions.html>`__.

For example, one can test a range of commits on a particular branch
since the branch was created::

        asv run mybranch@{u}..mybranch

.. note::

    Yes, this is git-specific for now.  Support for Mercurial or other
    DVCSes should be possible in the future, but not at the moment.

For example, to benchmark all of the commits since a particular tag
(``v0.1``)::

    asv run v0.1..master

In many cases, this may result in more commits than you are able to
benchmark in a reasonable amount of time.  In that case, the
``--steps`` argument is helpful.  It specifies the maximum number of
commits you want to test, and it will evenly space them over the
specified range.

You can benchmark all commits in the repository by using::

    asv run ALL

You may also want to benchmark every commit that has already been
benchmarked on all the other machines.  For that, use::

    asv run EXISTING

You can benchmark all commits since the last one that was benchmarked
on this machine.  This is useful for running in nightly cron jobs::

    asv run NEW

Finally, you can also benchmark all commits that have not yet been benchmarked
for this machine::

    asv run MISSING

The results are stored as a tree of files in the directory
``results/$MACHINE``, where ``$MACHINE`` is the unique machine name
that was set up in your ``~/.asv-machine.json`` file.  In order to
combine results from multiple machines, the normal workflow is to
commit these results to a source code repository alongside the results
from other machines.  These results are then collated and "published"
altogether into a single interactive website for viewing.

You can also continue to generate benchmark results for other commits,
or for new benchmarks and continue to throw them in the ``results``
directory.  **airspeed velocity** is designed from the ground up to
handle missing data where certain benchmarks have yet to be performed
-- it's entirely up to you how often you want to generate results, and
on which commits and in which configurations.

Viewing the results
-------------------

To collate a set of results into a viewable website, run::

    asv publish

This will put a tree of files in the ``html`` directory.  This website
can not be viewed directly from the local filesystem, since web
browsers do not support AJAX requests to the local filesystem.
Instead, **airspeed velocity** provides a simple static webserver that
can be used to preview the website.  Just run::

    asv preview

and open the URL that is displayed at the console.  Press Ctrl+C to
stop serving.

.. image:: screenshot.png

To share the website on the open internet, simply put these files on
any webserver that can serve static content.  Github Pages works quite
well, for example.  If using Github Pages, asv includes the
convenience command ``asv gh-pages`` to automatically publish the
results to the ``gh-pages`` branch.

Managing the results database
-----------------------------

The ``asv rm`` command can be used to remove benchmarks from the
database.  The command takes an arbitrary number of ``key=value``
entries that are "and"ed together to determine which benchmarks to
remove.

The keys may be one of:

- ``benchmark``: A benchmark name

- ``python``: The version of python

- ``commit_hash``: The commit hash

- machine-related: ``machine``, ``arch``, ``cpu``, ``os``, ``ram``

- environment-related: a name of a dependency, e.g. ``numpy``

The values are glob patterns, as supported by the Python standard
library module `fnmatch`.  So, for example, to remove all benchmarks
in the ``time_units`` module::

    asv rm "benchmark=time_units.*"

Note the double quotes around the entry to prevent the shell from
expanding the ``*`` itself.

The ``asv rm`` command will prompt before performing any operations.
Passing the ``-y`` option will skip the prompt.  Note that generally
the results will be stored in a source code repository, so it should
be possible to undo any of the changes using the DVCS directly as
well.

Here is a more complex example, to remove all of the benchmarks on
Python 2.7 and the machine named ``giraffe``::

    asv rm python=2.7 machine=giraffe


Finding a commit that produces a large regression
-------------------------------------------------

Since benchmarking can be rather time consuming, it's possible likely
that you're only benchmarking a subset of all commits in the
repository.  When you discover from the graph that the runtime between
commit A and commit B suddenly doubles, you don't know which
particular commit in that range is the likely culprit.  ``asv find``
can be used to help find a commit within that range that produced a
large regression using a binary search.  You can select a range of
commits easily from the web interface by dragging a box around the
commits in question.  The commit hashes associated with that range
is then displayed in the "commits" section of the sidebar.  We'll copy
and paste this commit range into the commandline arguments of the
``asv find`` command, along with the name of a single benchmark to use.
The output below is truncated to show how the search progresses::

    $ asv find 05d4f83d..b96fcc53 time_coordinates.time_latitude
    - Running approximately 10 benchmarks within 1156 commits
    - Testing <----------------------------O----------------------------->
    - Testing <-------------O-------------->------------------------------
    - Testing --------------<-------O------>------------------------------
    - Testing --------------<---O--->-------------------------------------
    - Testing --------------<-O->-----------------------------------------
    - Testing --------------<O>-------------------------------------------
    - Testing --------------<>--------------------------------------------
    - Greatest regression found: 2918f61e

The result, ``2918f61e`` is the commit found with the largest
regression, using the binary search.

.. note::

    The binary search used by ``asv find`` will only be effective when
    the runtimes over the range are more-or-less monotonic.  If there
    is a lot of variation within that range, it may find only a local
    maximum, rather than the global maximum.  For best results, use a
    reasonably small commit range.

.. profiling_

Running a benchmark in the profiler
-----------------------------------

**airspeed velocity** can oftentimes tell you *if* something got
slower, but it can't really tell you *why* it got slower.  That's a
where a profiler comes in.  **airspeed velocity** has features to
easily run a given benchmark in the Python standard library's
`cProfile` profiler, and then open the profiling data in the tool of
your choice.

``asv profile`` will profiles a given benchmark on a given revision of
the project.

.. note::

    You can also pass the ``--profile`` option to ``asv run``.  In
    addition to running the benchmarks as usual, it also generates
    them again in the `cProfile` profiler and save the results.  ``asv
    preview`` will use this data, if found, rather than needing to
    profile the benchmark each time.  However, it's important to note
    that profiler data contains absolute paths to the source code, so
    they are generally not portable between machines.

``asv profile`` takes as arguments the name of the benchmark and the
hash, tag or branch of the project to run it in.  Below is a real
world example of testing the ``astropy`` project.  By default, a
simple table summary of profiling results is displayed::

    > asv profile time_units.time_very_simple_unit_parse 10fc29cb

         8700042 function calls in 6.844 seconds

     Ordered by: cumulative time

     ncalls  tottime  percall  cumtime  percall filename:lineno(function)
          1    0.000    0.000    6.844    6.844 asv/benchmark.py:171(method_caller)
          1    0.000    0.000    6.844    6.844 asv/benchmark.py:197(run)
          1    0.000    0.000    6.844    6.844 /usr/lib64/python2.7/timeit.py:201(repeat)
          3    0.000    0.000    6.844    2.281 /usr/lib64/python2.7/timeit.py:178(timeit)
          3    0.104    0.035    6.844    2.281 /usr/lib64/python2.7/timeit.py:96(inner)
     300000    0.398    0.000    6.740    0.000 benchmarks/time_units.py:20(time_very_simple_unit_parse)
     300000    1.550    0.000    6.342    0.000 astropy/units/core.py:1673(__call__)
     300000    0.495    0.000    2.416    0.000 astropy/units/format/generic.py:361(parse)
     300000    1.023    0.000    1.841    0.000 astropy/units/format/__init__.py:31(get_format)
     300000    0.168    0.000    1.283    0.000 astropy/units/format/generic.py:374(_do_parse)
     300000    0.986    0.000    1.115    0.000 astropy/units/format/generic.py:345(_parse_unit)
    3000002    0.735    0.000    0.735    0.000 {isinstance}
     300000    0.403    0.000    0.403    0.000 {method 'decode' of 'str' objects}
     300000    0.216    0.000    0.216    0.000 astropy/units/format/generic.py:32(__init__)
     300000    0.152    0.000    0.188    0.000 /usr/lib64/python2.7/inspect.py:59(isclass)
     900000    0.170    0.000    0.170    0.000 {method 'lower' of 'unicode' objects}
     300000    0.133    0.000    0.133    0.000 {method 'count' of 'unicode' objects}
     300000    0.078    0.000    0.078    0.000 astropy/units/core.py:272(get_current_unit_registry)
     300000    0.076    0.000    0.076    0.000 {issubclass}
     300000    0.052    0.000    0.052    0.000 astropy/units/core.py:131(registry)
     300000    0.038    0.000    0.038    0.000 {method 'strip' of 'str' objects}
     300003    0.037    0.000    0.037    0.000 {globals}
     300000    0.033    0.000    0.033    0.000 {len}
          3    0.000    0.000    0.000    0.000 /usr/lib64/python2.7/timeit.py:143(setup)
          1    0.000    0.000    0.000    0.000 /usr/lib64/python2.7/timeit.py:121(__init__)
          6    0.000    0.000    0.000    0.000 {time.time}
          1    0.000    0.000    0.000    0.000 {min}
          1    0.000    0.000    0.000    0.000 {range}
          1    0.000    0.000    0.000    0.000 {hasattr}
          1    0.000    0.000    0.000    0.000 /usr/lib64/python2.7/timeit.py:94(_template_func)
          3    0.000    0.000    0.000    0.000 {gc.enable}
          3    0.000    0.000    0.000    0.000 {method 'append' of 'list' objects}
          3    0.000    0.000    0.000    0.000 {gc.disable}
          1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
          3    0.000    0.000    0.000    0.000 {gc.isenabled}
          1    0.000    0.000    0.000    0.000 <string>:1(<module>)

Navigating these sorts of results can be tricky, and generally you
want to open the results in a GUI tool, such as `RunSnakeRun
<http://www.vrplumber.com/programming/runsnakerun/>`__.  By passing
the ``--gui=runsnake`` to ``asv profile``, the profile is collected
(or extracted) and opened in the RunSnakeRun tool.

.. note::

    To make sure the line numbers in the profiling data correctly
    match the source files being viewed in RunSnakeRun, the correct
    revision of the project is checked out before opening it in the
    external GUI tool.

You can also get the raw profiling data by using the ``--output``
argument to ``asv profile``.

.. comparing_

Comparing the resuls for two revisions
--------------------------------------

In some cases, you may want to directly compare the results for two specific
revisions of the project. You can do so with the ``compare`` command::

    $ asv compare 7810d6d7 19aa5743
    · Fetching recent changes.

    All benchmarks:

        before     after       ratio
      [7810d6d7] [19aa5743]
    +    1.75ms   152.84ms     87.28  time_quantity.time_quantity_array_conversion
    +  933.71μs   108.22ms    115.90  time_quantity.time_quantity_init_array
        83.65μs    55.38μs      0.66  time_quantity.time_quantity_init_scalar
       281.71μs   146.88μs      0.52  time_quantity.time_quantity_scalar_conversion
    +    1.31ms     7.75ms      5.91  time_quantity.time_quantity_ufunc_sin
          5.73m      5.73m      1.00  time_units.mem_unit
    ...

This will show the times for each benchmark for the first and second revision,
and the ratio of the second to the first. In addition, the benchmarks will be
color coded green and red if the bechmark improves or worsens more than a
certain threshold factor, which defaults to 2 (that is, benchmarks that improve
by more than a factor of 2 or worsen by a factor of 2 are color coded). The
threshold can be set with the ``--threshold=value`` option. Finally, the
benchmarks can be split into ones that have improved, stayed the same, and
worsened, using the same threshold.
