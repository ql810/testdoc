.. _add_kconfig_entry:

=================
Add KConfig Entry
=================

.. note::
   Reference :ref:`kconfig` documentation for syntax and examples.

* Each folder should have a **Kconfig** file so that KConfig program can traverse the directory tree.
  It is possible by using :ref:`kconfig_source` to point to **Kconfig** files in sub-directories.

* KConfig symbol must **NOT** have ``CONFIG_`` prefix. CMake script inserts this prefix to create preprocessor symbols.

  Example: ``MY_SYMBOL_0`` KConfig symbol becomes ``CONFIG_MY_SYMBOL_0`` preprocessor symbol.

  .. important::
     CMake script also makes these preprocessor symbols available to source code.

* Entries in existing Kconfig files are also good examples. KConfig language is very easy to understand.
