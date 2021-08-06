.. _interfacing:

Interfacing with ABRT
=====================

.. _socketapi:

Socket API
----------

Socket API allows creation of new problems. It is used by Python and Java hooks
to pass required data to ``abrtd``.

Socket path::

        /var/run/abrt/abrt.socket

First line has to contain HTTP header::

        POST / HTTP/1.1\r\n\r\n

followed by ``key=value`` pairs delimited with ``\0``.
The server expects another ``\0`` at the end of the message.

Mandatory keys:

* ``type`` — *(string)* problem type, see :ref:`problemtypes`.
* ``pid`` — *(integer)* process ID of the crashed procss, ranges from 0 to `PID_MAX`
  (``/proc/sys/kernel/pid_max``)
* ``executable`` — *(string)* path of the affected executable
* ``backtrace`` — *(string)*
* ``reason`` — *(string)* reason of the crash

To ensure the problem can be reported to Bugzilla via report-gtk or
report-cli you have to add the following keys with the following contents:

* ``duphash`` — *(string)* duplicate hash. The hash is placed in Bugzilla's
  ``Whiteboard`` field in the format ``abrt_hash:$duphash``. For C/C++, the content
  of ``duphash`` is the SHA-1 digest of the concatenation of the names of top six
  functions on the stacktrace. For Python exceptions, it is the SHA-1 digest of the
  stacktrace.
* ``uuid`` — *(string)* local identifier of the problem. The content can be the same
  as for ``duphash``.

Optionally, the server accepts other elements listed in :ref:`elements`.

If there's no error, the server responds with::

        HTTP/1.1 201 Created\r\n\r\n

or ``400`` status code in case of error.

:ref:`pyhook` may serve as an example of the socket API usage.

.. _dbusapi:

DBus API
--------

Documentation for the DBus API at ``org.freedesktop.problems`` is available as part
of the ``abrt-dbus`` package or online at
http://jfilak.fedorapeople.org/ProblemsAPI/re01.html.

.. _pythonapi:

Python API
----------

Documentation for the Python API is available in the ``python3-abrt(5)`` man page
(part of the ``python3-abrt`` package).
