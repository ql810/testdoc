.. _kconfig:

=======
KConfig
=======

Introduction
============

A collection of configuration options organized in a tree structure. Every entry has its own dependencies. These dependencies are used to determine the visibility of an entry. Any child entry is only visible if its parent entry is also visible.

The Language
============

* :ref:`kconfig_mainmenu`.

* :ref:`kconfig_config`

* :ref:`kconfig_choice_endchoice`

* :ref:`kconfig_menu_endmenu`

* :ref:`kconfig_source`

* :ref:`kconfig_comment`

* :ref:`kconfig_if_elseif_else_endif`

.. _kconfig_mainmenu:

The Syntax
==========

mainmenu
********

This sets title bar for config program. It is defined in the root Kconfig file.

.. code-block:: none

   mainmenu <"main_menu_title_text">

Examples:

   .. code-block:: none

      mainmenu "Example Main Menu"

.. _kconfig_config:

config
******

This defines a config symbol <symbol>.

.. code-block:: none

   config <symbol>
       <config_type ["config_entry_display_text"]>
       [depends on <condition>]
       [default <default_value>]
       [help <help_text>]

* ``config_type`` can be ``bool``,  ``int`` or ``string``.
* If ``config_entry_display_text`` is provided, user can modify the symbol value through GUI of config program.
  If not provided, the symbol won't be visible to user.
* ``condition`` can be another symbol or logic expression of multiple symbols.
  Such as ``depends on SYMBOL_X`` or ``depends on SYMBOL_A || SYMBOL_B`` or ``depends on SYMBOL_1 && SYMBOL2``.
  The symbol becomes unavailable to config program if ``condition`` does not meet.
* ``default`` declares a default value for the symbol.
* ``default_value``:

  -  ``y`` or ``n`` for ``bool``.
  - a number for ``int``.
  - a string for ``string``.
* ``help_text`` provides description/usage/explaination/etc to config program to display help to user regarding the symbol.

Examples:

   .. code-block:: none

      config ARCH_ARM
          bool

   .. code-block:: none

      config LED
          bool "LED"
          default n
          help
              Select to enable LED support.

   .. code-block:: none

      config ACCELEROMETER_LIS2DH
          bool "ST LIS2DH"
          depends on I2C || SPI
          default n
          help
              Select to enable ST LIS2DH accelerometer support.

   .. code-block:: none

      config SYS_LOG_DEFAULT_LEVEL
          int "Default level"
          depends on SYS_LOG
          range 0 4
          default 4
          help
              0 Disabled.
              1 ERROR.
              2 ERROR, WARNING.
              3 ERROR, WARNING, INFO.
              4 ERROR, WARNING, INFO, DEBUG.

   .. code-block:: none

      config SYS_LOG_BUFFER_SIZE
          int "Buffer size"
          depends on SYS_LOG
          default 1024
          help
              Number of bytes for syslog internal buffer.

.. _kconfig_choice_endchoice:

choice/endchoice
****************

Create a multiple choice menu.

.. code-block:: none

   choice <"choice_entry_text">
       <default <default_value>>
       [help <help_text>]

       <config 0>
       <config 1>
       ...
       <config n>
   endchoice

* ``default`` declares a default value for the choice menu.
* ``default_value`` must be one of the config symbols within this choice list.
* ``help_text`` provides description/usage/explaination/etc to config program to display help to user regarding the choice menu.

Examples:

   .. code-block:: none

      choice
          prompt "Optimization level"
          default OPTIMIZATION_NONE
          help
              Control compiler optimization level.

          config OPTIMIZATION_NONE
              bool "No optimizations"
              help
                  Compiler optimizations will be set to -O0.

          config OPTIMIZATION_DEBUG
              bool "Optimizations for debugging"
              help
                  Compiler optimizations will be set to -Og.

          config OPTIMIZATION_SIZE
              bool "Optimizations for size"
              help
                  Compiler optimizations will be set to -Os.

          config OPTIMIZATION_SPEED
              bool "Optimizations for speed"
              help
                  Compiler optimizations will be set to -O2.
      endchoice

.. _kconfig_menu_endmenu:

menu/endmenu
************

This groups multiple entries in a menu structure. Entries could be ``config`` and/or sub-menu (``menu/endmenu``, ``choice/endchoice``).

.. code-block:: none

   menu <"menu_entry_text">
       <entry 0>
       <entry 1>
       ...
       <entry n>
   endmenu

Examples:

   .. code-block:: none

      menu "Debug Options"
          menu "Logging Options"
              config SYS_LOG
                  bool "Enable system logging (SYSLOG)"
                  default n
                  help
                      Turn on/off system logging.

              config SYS_LOG_DEFAULT_LEVEL
                  int "Default level"
                  depends on SYS_LOG
                  range 0 4
                  default 4
                  help
                      0 Disabled.
                      1 ERROR.
                      2 ERROR, WARNING.
                      3 ERROR, WARNING, INFO.
                      4 ERROR, WARNING, INFO, DEBUG.

              config SYS_LOG_BUFFER_SIZE
                  int "Buffer size"
                  depends on SYS_LOG
                  default 1024
                  help
                      Number of bytes for syslog internal buffer.

              config SYS_LOG_ENABLE_TIMESTAMP
                  bool "Enable timestamp"
                  depends on SYS_LOG
                  default n
          endmenu

          config CRASH_DUMP_SUPPORT
              bool "Crash Dump Support"
              default y

      endmenu

.. _kconfig_source:

source
******

This imports another Kconfig file.

.. code-block:: none

   source <"kconfig_file">

* ``kconfig_file`` must include path relative to root Kconfig file.

Examples:

   .. code-block:: none

      source "drivers/Kconfig"


.. _kconfig_comment:

comment
*******

This defines a comment that can be displayed in GUI of config program.

.. code-block:: none

   comment <"comment_text">
       [depends on <condition>]

* ``condition`` can be another symbol or logic expression of multiple symbols.
  Such as ``depends on SYMBOL_X`` or ``depends on SYMBOL_A || SYMBOL_B`` or ``depends on SYMBOL_1 && SYMBOL2``.
  The comment becomes unavailable to config program if ``condition`` does not meet.

Examples:

   .. code-block:: none

      comment "The following parameters are optional."

      config PARAMETER_A
          bool "Parameter A"
          default n

      config PARAMETER_B
          bool "Parameter B"
          default n


.. _kconfig_if_elseif_else_endif:

if/elseif/else/endif
********************

This defines an if block.

.. code-block:: none

   if <logic_expression>
       <true_if_block>
   else
       <false_if_block>
   endif

Examples:

   .. code-block:: none

      if (ARCH_ARM)
          source "arch/arm/Kconfig"
      endif
