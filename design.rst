.. _design:

Design
======

.. _abrtd:

Daemon
------

``abrtd`` is a daemon that watches for application crashes. When a crash occurs,
it collects the problem data (core file, application's command line, ...) and takes
action according to the configuration and the type of application that crashed.

By default it uses inotify interface [#inotify]_ to monitor the dump location
(``/var/spool/abrt/``) for new directories created by C/C++ hook and a :ref:`socketapi`
(``/var/run/abrt/abrt.socket``) used by other hooks like :ref:`pyhook`.

The reason for using socket instead of direct filesystem access is security.
When a Python script throws unhandled exception, :ref:`pyhook` catches it, running
as a part of the broken Python application. The application is running
with certain SELinux privileges, for example it can not execute other
programs, or to create files in ``/var/spool/abrt`` or anything else required
to properly fill a problem directory. Adding these privileges to every
application would weaken the security.
The most suitable solution for the Python application is
to open a socket where ``abrtd`` is listening, write all relevant
data to that socket, and close it. ``abrtd`` handles the rest of the processes.

.. _pyhook:

Python hook
-----------

The ``python3-abrt-addon`` package provides an exception handler for Python 3
applications.

The Python interpreter automatically imports the ``abrt.pth`` file installed in
``/usr/lib64/python3.7/site-packages/``. This file in turn imports
``abrt_exception_handler.py`` which overrides Python's default ``sys.excepthook``
with a custom handler that forwards unhandled exceptions to ``abrtd`` via its
:ref:`socketapi`.

Automatic import of site-specific modules can be disabled by passing the ``-S``
option to the Python interpreter::

        python -S file.py



.. _oopswatcher:

Oops watcher
------------

Kernel oopses are detected by watcher ``abrt-dump-journal-oops``, typicaly this
process runs as a daemon and watches systemd-journal. When kernel oops logs
appears, watcher extracts them and creates problem dir, which is further
processed by post-create event handler for type Kerneloops.

.. _xorgwatcher:

Xorg watcher
------------

Xorg crashes are detected by watcher ``abrt-dump-journal-xorg``. Mechanism is
same as in oops watcher, systemd-journal is watched and Xorg crashes are
extracted in time of their occurence. In addition xorg watcher can be
configured to search for next Xorg crashes, config file is located in
``/etc/abrt/plugins/xorg.conf``.

.. _eventdesign:

Events
------

A problem life cycle is driven by events in ABRT. For example:

* `Event 1` — a problem data directory is created.

* `Event 2` — problem data is analyzed.

* `Event 3` — a problem is reported to Bugzilla.

When a problem is detected and its defining data is stored,
the problem is processed by running events on the problem's data directory.
For event configuration how-to, refer to :ref:`eventconf`.

Standard ABRT installation currently supports several default
events that can be selected and used during problem reporting process.
Refer to :ref:`standardevents` to see the list of these events.

Only following three events are run automatically by ABRT:

``post-create``
        runs after the problem directory creation

``notify``
        runs after the processing chain is finished to notify user about new problem

``notify-dup``
        similar to ``notify`` for duplicate problems. See :ref:`dedup`.

.. _dedup:

Deduplication
-------------

When ABRT catches new crash it compares it to the rest of the stored problems
to avoid storing duplicate crashes.

It first checks if there is ``core_bactrace`` or ``uuid`` item in the problem
directory we are processing.

If there is a ``core_backtrace``, it iterates over all other dump
directories and computes similarity to their core backtraces (if any).
If one of them is similar enough to be considered duplicate, event processing
is stopped and only ``notify-dup`` event is fired.

If there is an ``uuid`` item (and no core backtrace), simple comparison
of ``uuid`` hashes is used for duplicate detection.

.. _elements:

Elements collected by ABRT
--------------------------

Commonly available elements:

===================== ======================================================== ====================
Property              Meaning                                                  Example
===================== ======================================================== ====================
``executable``        Executable path of the component which caused the        ``'/usr/bin/time'``
                      problem.  Used by the server to determine
                      ``component`` and ``package`` data.
``type``              Problem typem, see :ref:`problemtypes`.                  ``'Python'``
``component``         Component which caused this problem.                     ``'time'``
``hostname``          Hostname of the affected machine.                        ``'fiasco'``
``os_release``        Operating system release string.                         ``'Fedora release 17 (Beefy Miracle)'``
``uid``               User ID                                                  ``1000``
``username``          User name                                                ``'jeff'``
``architecture``      Machine architecture string                              ``'x86_64'``
``kernel``            Kernel version string                                    ``'3.6.6-1.fc17.x86_64'``
``package``           Package string                                           ``'time-1.7-40.fc17.x86_64'``
``time``              Time of the occurrence (unixtime)                         ``datetime.datetime(2012, 12, 2, 16, 18, 41)``
``count``             Number of times this problem occurred                     ``1``
``pkg_name``          Package name                                             ``'time'``
``pkg_epoch``         Package epoch                                            ``0``
``pkg_version``       Package version                                          ``'1.7'``
``pkg_release``       Package release                                          ``'40.fc17'``
``pkg_arch``          Package architecture                                     ``'x86_64'``
``uuid``              Unique problem identifier computed as a hash of the
                      first three frames of the backtrace                      ``'c55e3deb95d46553fdbefb1bc1d020e89a762fb7'``
===================== ======================================================== ====================

Elements dependent on problem type:

===================== ====================================================================== ====================================== ===============================
Property              Meaning                                                                Example                                Applicable
===================== ====================================================================== ====================================== ===============================
``abrt_version``      ABRT version string                                                    ``'2.0.18.84.g211c'``                  Crashes caught by ABRT
``core_backtrace``    Machine readable backtrace with no private data                                                               Python, Ruby, Kerneloops
``backtrace``         Original backtrace or backtrace produced by retracing                                                         Python, Ruby, Xorg, Kerneloops
                      process
``dso_list``          List of dynamic libraries loaded at the time of crash                                                         Python
``cmdline``           Copy of ``/proc/<pid>/cmdline`` file                                   ``'/usr/bin/gtk-builder-convert'``     Python, Ruby, Kerneloops
``environ``           Runtime environment of the process                                                                            Python
``pid``               Process ID                                                             ``'42'``                               Python, Ruby
``suspend_stats``     Copy of ``/sys/kernel/debug/suspend_stats``                                                                   Kerneloops
``reported_to``       If the problem was already reported, this item contains                                                       Reported problems
                      URLs of the services where it was reported
``event_log``         ABRT event log                                                                                                Reported problems
``dmesg``             Copy of ``dmesg``                                                                                             Kerneloops
===================== ====================================================================== ====================================== ===============================


.. _problemtypes:

Supported problem types
^^^^^^^^^^^^^^^^^^^^^^^

Supported values for ``type`` element:

* ``java``
* ``Kerneloops``
* ``selinux``
* ``Python``
* ``Python3``
* ``Ruby``
* ``xorg``

.. rubric:: Footnotes

.. [#procfs] https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/plain/Documentation/filesystems/proc.txt
.. [#corepattern] https://www.kernel.org/doc/html/latest/admin-guide/sysctl/kernel.html#core-pattern
.. [#inotify] https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/plain/Documentation/filesystems/inotify.txt
