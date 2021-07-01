.. _developer:

Documentation for developers
============================

Contributing
------------

Send us patches to our mailing list at
http://lists.fedorahosted.org/mailman/listinfo/crash-catcher

or create a pull request for the corresponding repository
in `the abrt GitHub organization <https://github.com/abrt>`_.


Where appropriate, your submission should come with a test —
either unit test or as a part of our integration test suite.
See :ref:`newinttest` for more details.


Nightly builds
--------------

Nightly builds and repositories for Fedora and RHEL
are avaialable at https://copr.fedorainfracloud.org/groups/g/abrt/coprs/

Ignoring common functions on the stack
--------------------------------------

To improve clustering of similar crashes done by
:ref:`analytics`, backtraces are first normalized to skip
common functions like ``_start`` from glibc or
``__kernel_vsyscall`` from Linux kernel.

Such functions are listed in
`satyr/lib/normalize.c <https://github.com/abrt/satyr/blob/master/lib/normalize.c>`_.

Writing man pages
-----------------

Abrt man pages are written in AsciiDoc, a minimal text markup language.
The ``asciidoctor`` tool generates the formatted manpage files.
Man pages in AsciiDoc format are stored in each project's ``doc/`` directory.
For example, man pages for ``libreport`` are placed in
`libreport/doc/ <https://github.com/abrt/libreport/blob/master/doc/>`_.

See `the Asciidoctor documentation <https://docs.asciidoctor.org/asciidoctor/latest/manpage-backend/>`_
for more information on writing man pages in AsciiDoc.
The `AsciiDoc Writer's Guide <https://asciidoctor.org/docs/asciidoc-writers-guide/>`_
is a good resource for learning AsciiDoc syntax,
although not all features are supported by the man page generator.

The existing files in `abrt/doc/ <https://github.com/abrt/abrt/blob/master/doc/>`_
can be used as templates for creating new man pages.
