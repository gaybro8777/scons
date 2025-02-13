Contributing to SCons
#####################

Introdction
===========

Thanks for taking the time to contribute!

This page will talk a bit about the development process,
and about the SCons tree (right here, if you're reading this
in Github, or in a cloned repository).

There are lots of places we could use help - please don't
think we're only interested in contributions to code.

If you're going to contribute, we'd love to get to know you
a bit and understand and what problems you're looking to solve,
or what you are intending to improve, whether that's documnetation,
code, examples, tutorials, etc. A great way to introduce yourself is to
to hop onto the `SCons Discord Server <https://discord.gg/bXVpWAy>`_
to chat.  You don't have to be a regular Discord user,
there is a web interface.

Resources
=========

Here are some important pointers:

  * `SCons Home Page <https://scons.org>`_
  * `Github Project Page <https://github.com/scons/scons>`_
  * `Bugs and Feature Requests <https://scons.org/bugs.html>`_
  * `Development <https://scons.org/dev.html>`_
  * `SCons Developer's Guidelines <https://scons.org/guidelines.html>`_
  * `Contacts <(https://scons.org/contact.html>`_

Reporting Bugs
==============

One of the easiest ways to contribute is by filing bugs.
The SCons project welcomes bug reports and feature requests,
but we have do a preference for having talked about it first -
we request you send an email to the 
`SCons Users Mailing List <https://two.pairlist.net/mailman/listinfo/scons-users>`_
or hop on the Discord channel (see link above), and if so
instructed, then proceed to an issue report.

You can explore the list of existing bugs on GitHub.
Sometimes there's work in progress which may include temporary
workarounds for the problem you've run into::

    https://github.com/SCons/scons/issues


About the Development Tree
==========================

This tree has more logic than just the SCons code itself -
there's quite a bit to do with packaging it in a couple
of forms: a Python-installable package (source distribution
and installable wheel file), a portable zip (or tar) distribution
called "scons-local", and a full source bundle.  You usually
don't need to worry about the packaging methods when working
on a source or doc contribution.

To the extent that this tree is about building SCons packages, the *full*
development cycle is not just to test the code directly, but to package SCons,
unpack the package, "install" SCons in a test subdirectory, and then to run
the tests against the unpacked and installed software.  This helps eliminate
problems caused by, for example, failure to update the list of files to be
packaged.

For just working on making an individual change to the SCons source, however,
you don't actually need to build or install SCons; you *can* actually edit and
execute SCons in-place.  See the following sections below for more
information:

    `Making Changes`_
        How to edit and execute SCons in-place.

    `Debugging`_
        Tips for debugging problems in SCons.

    `Testing`_
        How to use the automated regression tests.

    `Typical Development Workflow`_
        An example of how to put the edit/execute/test pieces
        together in a reasonable development workflow.


Executing SCons Without Installing
==================================

Being a project written in Python, you don't have to "build"
SCons to run it, "build" actually refers to the steps taken to prepare
for constructing packages, the most time-consuing of which is
building the documentation.  

Documentation is written in a markup language which is a
light extension of Docbook-XML, and the doc build consists
of translating to pure docbook, then using standard tools to
generate HTML and PDF outputs from that. There's lots more
on the documentation process at the Documentation Toolchain page::

    https://github.com/SCons/scons/blob/master/doc/overview.rst


You can execute SCons directly from this repository. For Linux or UNIX::

    $ python scripts/scons.py [arguments]

Or on Windows::

    C:\scons>py scripts\scons.py [arguments]

If you run SCons this way, it will read the ``SConstruct`` file in this repo,
which will build and package SCons itself, which you probably don't want
unless you are doing development that relates to the packaging.

Often when doing SCons development, you have a sample project that
isn't working right, and it's useful to be able to build that
project from place you're working on changes to SCons. You
can do that with the ``C`` (change directory) option::

    $ python scripts/scons.py -C /some/other/location [arguments]

Other Required Software
=======================

Running SCons has no installation dependencies beyond a compatible version
of Python. The tools which will be used to to actually construct the
project, such as compilers, documentation production tools, etc.
should of course be installed by the appropriate means.  In order
to develop SCons and run its test suite, there are some dependencies,
listed in the ``requirements.txt`` file.

Making Changes
==============

Virtually all of the SCons functionality exists in the "build engine," the
``SCons`` subdirectory hierarchy that contains all of the modules that
make up SCons.  The ``scripts/scons.py`` wrapper script exists mainly to find
the appropriate build engine library and then execute it.

In order to make your own changes locally and test them by hand, simply edit
modules in the local ``SCons`` subdirectory tree and then run
(see the section above about `Executing SCons Without Installing`_)::

    $ python scripts/scons.py [arguments]

If you want to be able to just execute your modified version of SCons from the
command line, you can make it executable and add its directory to your $PATH
like so::

    $ chmod 755 scripts/scons.py
    $ export PATH=$PATH:`pwd`/scripts

You should then be able to run this version of SCons by just typing ``scons.py``
at your UNIX or Linux command line.

Note that the regular SCons development process makes heavy use of automated
testing.  See the `Testing`_ and `Typical Development Workflow`_ sections below for more
information about the automated regression tests and how they can be used in a
development cycle to validate that your changes don't break existing
functionality.


Debugging
=========

Python comes with a good interactive debugger.  When debugging changes by hand
(i.e., when not using the automated tests), you can invoke SCons under control
of the Python debugger by specifying the --debug=pdb option::

    $ scons --debug=pdb [arguments]
    > /home/knight/scons/SCons/Script/Main.py(927)_main()
    -> default_warnings = [ SCons.Warnings.CorruptSConsignWarning,
    (Pdb)

Once in the debugger, you can set breakpoints at lines in files in the build
engine modules by providing the path name of the file relative to the
top directory (that is, including the SCons/ as the first directory
component)::

    (Pdb) b SCons/Tool/msvc.py:158

Since Python 3.7.0 you can also insert a call to the ``breakpoint()``
function in your code, call ``scons.py`` normally, and it will drop into
the debugger at that point.

The debugger supports single stepping, stepping into functions, printing
variables, etc.

Trying to debug problems found by running the automated tests (see the
`Testing`_ section, below) is more difficult, because the test automation
harness re-invokes SCons and captures output. Consequently, there isn't an
easy way to invoke the Python debugger in a useful way on any particular SCons
call within a test script.

The most effective technique for debugging problems that occur during an
automated test is to use the good old tried-and-true technique of adding
statements to print tracing information.  But note that you can't just use
the ``print`` function, or even ``sys.stdout.write()`` because those change the
SCons output, and the automated tests usually look for matches of specific
output strings to decide if a given SCons invocation passes the test -
so these additions may cause apparent failures different than the one you
are trying to debug.

To deal with this, SCons supports a ``Trace()`` function that (by default) will
print messages to your console screen (``/dev/tty`` on UNIX or Linux, ``con`` on
Windows).  By adding ``Trace()`` calls to the SCons source code::

    def sample_method(self, value):
        from SCons.Debug import Trace
        Trace('called sample_method(%s, %s)\n' % (self, value))

You can then run automated tests that print any arbitrary information you wish
about what's going on inside SCons, without interfering with the test
automation.

The ``Trace()`` function can also redirect its output to a file, rather than the
screen::

    def sample_method(self, value):
        from SCons.Debug import Trace
        Trace('called sample_method(%s, %s)\n' % (self, value),
              file='trace.out')

Where the ``Trace()`` function sends its output is stateful: once you use the
``file=`` argument, all subsequent calls to ``Trace()`` send their output to the
same file, until another call with a ``file=`` argument is reached.


Testing
=======

Tests are run by the ``runtest.py`` script in the top directory.

There are two types of tests in this package:

1. Unit tests for individual SCons modules live underneath the SCons
   subdirectory and have the same base name as the module with ``Tests.py``
   appended--for example, the unit test for the ``Builder`` module in
   ``Builder.py`` is the ``BuilderTests.py`` script.

2. End-to-end tests of SCons live in the ``test/`` subdirectory.

You may specifically list one or more tests to be run::

        $ python runtest.py SCons/BuilderTests.py

        $ python runtest.py test/option-j.py test/Program.py

You also use the ``-f`` option to execute just the tests listed in a specified
text file::

        $ cat testlist.txt
        test/option-j.py
        test/Program.py
        $ python runtest.py -f testlist.txt

One test must be listed per line, and any lines that begin with '#' will be
ignored (allowing you, for example, to comment out tests that are currently
passing and then uncomment all of the tests in the file for a final validation
run).

The runtest.py script also takes a ``-a`` option that searches the tree for all of
the tests and runs them::

        $ python runtest.py -a

If a previous run had test failures, those are saved to logfile which
can be used to run just the failed tests - this is useful for the common
case of a change breaking a few things, and you want to first check that
a fix fixes those, before rerunning the full suite::

        $ python runtest.py --retry

If more than one test is run, the ``runtest.py`` script prints a summary of
any tests that failed or yielded no result (usually these are skips due
to run-time checks of conditions). ``runtest.py`` has options to change
the output, just see the command's help message.

The above invocations all test directly the files underneath the ``SCons/``
subdirectory, and do not require that a build be performed first.

Typical Development Workflow
============================

    Caveat: The point of this section isn't to describe one dogmatic workflow.
    Just running the test suite can be time-consuming, and getting a patch to
    pass all of the tests can be more so.  If you're genuinely blocked, it may
    make more sense to submit a patch with a note about which tests still
    fail, and how.  Someone else may be able to take your "initial draft" and
    figure out how to improve it to fix the rest of the tests.  So there's
    plenty of room for use of good judgement.

The various techniques described in the above sections can be combined to
create simple and effective workflows that allow you to validate that patches
you submit to SCons don't break existing functionality and have adequate
testing, thereby increasing the speed with which they can be integrated.

For example, suppose your project's SCons configuration is blocked by an SCons
bug, and you decide you want to fix it and submit the patch.  Here's one
possible way to go about doing that (using UNIX/Linux as the development
platform, Windows users can translate as appropriate)):

- Change to the top of your checked-out SCons tree.

- Confirm that the bug still exists in this version of SCons by using the ``-C``
  option to run the broken build::

      $ python scripts/scons.py -C /home/me/broken_project .

- Fix the bug in SCons by editing appropriate module files underneath
  SCons.

- Confirm that you've fixed the bug affecting your project::

      $ python scripts/scons.py -C /home/me/broken_project .

- Test to see if your fix had any unintended side effects that break existing
  functionality::

      $ python runtest.py -a -o test.log

  Be patient, there are more than 1100 test scripts in the whole suite!

  If any test scripts fail, they will be listed in a summary at the end of the
  log file.  Some test scripts may also report NO RESULT because (for example)
  your local system is the wrong type or doesn't have some installed utilities
  necessary to run the script.  In general, you can ignore the NO RESULT list,
  beyond having checked once that the tests that matter to your change are
  actually being executed on your test system!  These failed tests are
  automatically saved to ``failed_tests.log``.

- Now debug the test failures and fix them, either by changing SCons, or by
  making necessary changes to the tests (if, for example, you have a strong
  reason to change functionality, or if you find that the bug really is in the
  test script itself).  After each change, use the ``--retry``
  option to examine the effects of the change on the subset of tests that
  last failed::

      $ [edit]
      $ python runtest.py --retry

  Repeat this until all of the tests that originally failed now pass.

- Now you need to go back and validate that any changes you made while getting
  the tests to pass didn't break the fix you originally put in, and didn't
  introduce any *additional* unintended side effects that broke other tests::

      $ python scripts/scons.py -C /home/me/broken_project .
      $ python runtest.py -a -o test.log

Of course, the above is only one suggested workflow.  In practice, there is a
lot of room for judgment and experience to make things go quicker.  For
example, if you're making a change to just the Java support, you might start
looking for regressions by just running the ``test/Java/\*.py`` tests instead of
running all tests with ``runtest.py -a``.


Building Packages
=================

We use SCons (version 3.1.2 or later) to build its own packages.  If you
already have an appropriate version of SCons installed on your system,
you can build everything by simply running it::

    $ scons

If you don't have SCons already installed on your system,
you can run the build directly from the source tree
(see the section above about `Executing SCons Without Installing`_)::

    $ python scripts/scons.py

Those are full builds: depending on the utilities installed on your system,
any or all of the following packages will be built::

    SCons-4.3.0-py3-none-any.whl
    SCons-4.3.0ayyyymmdd.tar.gz
    SCons-4.3.0ayyyymmdd.zip
    scons-doc-4.3.0ayyyymmdd.tar.gz
    scons-local-4.3.0ayyyymmdd.tar.gz
    scons-local-4.3.0ayyyymmdd.zip

The ``SConstruct`` file is supposed to be smart enough to avoid trying to build
packages for which you don't have the proper utilities installed.

If you receive a build error, please report it to the scons-devel mailing list
and open a bug report on the SCons bug tracker.

Note that in addition to creating the above packages, the default build will
also unpack one or more of the packages for testing.

If you're working on documentation and just want to make sure that still builds,
there's a "doc" target::

    $ python scripts/scons.py doc

Contents of this Tree
=====================

Not guaranteed to be up-to-date (but better than nothing):

bench/
    A subdirectory for benchmarking scripts, used to perform timing tests
    to decide what specific idioms are most efficient for various parts of
    the code base.  We check these in so they're available in case we have
    to revisit any of these decisions in the future.

bin/
    Miscellaneous utilities used in SCons development.  Right now,
    some of the stuff here includes:

    - a script that runs pychecker on our source tree;

    - a script that counts source and test files and numbers of lines in each;

    - a prototype script for capturing sample SCons output in xml files;

    - a script that can profile and time a packaging build of SCons itself;

    - a copy of xml_export, which can retrieve project data from SourceForge;
      (obsolete, as project now lives on GitHub and PyPi).

    - scripts and a Python module for translating the SCons home-brew XML
      documentation tags into DocBook and man page format

bootstrap.py
    Obsolete packaging logic - ignore this.

debian/
    Files needed to construct a Debian package. The contents of this directory
    are dictated by the
    `Debian Policy Manual <https://www.debian.org/doc/debian-policy>`).
    The package will not be accepted into the Debian distribution unless
    the contents of this directory satisfy the relevant Debian policies.
    At this point, this is a sample; SCons is packaged in the Debian
    project (and thus inherited by projects which derive from it, if
    they haven't made their own packages). See:

    - <https://packages.debian.org/bullseye/scons>_
    - <https://packages.ubuntu.com/impish/scons>_

doc/
    SCons documentation.  A variety of things here, in various stages of
    (in)completeness. Note not all of the documentation is in ``doc`` -
    for tools and other self-contained items, there is often a documentation
    file together with the source, with a ``.xml`` suffix, in the same
    way there is often a unit-test file kept together with the source it tests.

LICENSE
    A copy of the copyright and terms under which SCons is distributed (the
    Open Source Initiative-approved MIT license).

LICENSE-local
    A copy of the copyright and terms under which SCons is distributed for
    inclusion in the scons-local-{version} packages.  This is the same as
    LICENSE with a preamble that specifies the licensing terms are for SCons
    itself, not any other package that includes SCons.

README.rst
    What you're looking at right now.

README-local.rst
    A README file for inclusion in the scons-local-{version} packages.
    Similar to this file, but stripped down and modified for people looking at
    including SCons in their shipped software.

README-SF.rst
    A README file the SourceForge project page - although the project is
    no longer developed on SourceForge, this still serves as a download
    location.

runtest.py
    Script for running SCons tests.  By default, this will run a test against
    the code in the local SCons tree, so you don't have to do a build before
    testing your changes.

SConstruct
    The file describing to SCons how to build the SCons distribution.

    (It has been pointed out that it's hard to find the SCons API in this
    SConstruct file, and that it looks a lot more like a pure Python script
    than a build configuration file.  That's mainly because all of the magick
    we have to perform to deal with all of the different packaging formats
    requires a lot of pure Python manipulation.  In other words, don't look at
    this file for an example of how easy it is to use SCons to build "normal"
    software.)

SCons/
    Where the actual source code is kept, of course.

test/
    End-to-end tests of the SCons utility itself.  These are separate from the
    individual module unit tests, which live side-by-side with the modules
    under SCons.

testing/
    SCons testing framework.
