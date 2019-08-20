ABRT project documentation
==========================

.. image:: https://readthedocs.org/projects/abrt/badge/?version=latest
:target: https://abrt.readthedocs.io/en/latest/?badge=latest
:alt: Documentation Status

Requirements
------------

- ``python-sphinx`` >= 1.1

Building
--------

``make html``
    for building html version
``make watch``
    to build continually on source code changes
    (requires ``inotifywait`` from ``inotify-tools`` package)
