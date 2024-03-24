.. _unit_test:

=========
Unit Test
=========

Brief
=====

- Introduction to unit tests with ThrowTheSwitch Unity.
- Project has test/mock auto-generator to simplify test/mock implementation. 

Create Test Files
=================

Only need to add test functions to test files. Unity test harness is handled by the generator.
Test files can have mocks.

Test File Without Mocks
-----------------------

This is a skeleton of a test file without mocks.
Test function must always start with ``test_``  in order for test generator to recognize it.

   .. code-block:: console

      #include <unity.h>
      <add include of lib under test here>

      void test_TEST_DESCRIPTION(void)
      {
          <add test code here>

          /* assert */
          <add test qualification here>
      }

   Example of ``add`` unit test.

   .. code-block:: console

      #include <unity.h>
      #include "add.h"

      void test_add_test0(void)
      {
          int value = add(1, 2);

          /* assert */
          TEST_ASSERT_EQUAL(3, value);
      }

Test File With Mocks
--------------------

This is a skeleton of a test file with mocks.
Test function must always start with ``test_``  in order for test generator to recognize it.
Mock header files always start with ``mock_``.

.. code-block:: console

   #include <unity.h>
   <add include of mock headers here>
   <add include of lib under test here>

   void test_TEST_DESCRIPTION(void)
   {
       <add test code here>

       /* assert */
       <add test qualification here>
   }

Example of ``get_error`` unit test with mocks.

.. code-block:: console

   #include <unity.h>
   #include "mock_get_last_error_value.h"              /* This file is auto-generated */
   #include "get_error.h"

   void test_get_error_test1(void)
   {
       const char * error_str;

       get_last_error_value_ExpectAndReturn(1234);     /* This is a mock for get_last_error_value() */

       error_str = get_last_error_string();

       /* assert */
       TEST_ASSERT_EQUAL_STRING("Unknown Error.", error_str);
   }


Add Tests to Project
====================

#. Create test file.

#. Create folder named ``mock`` for all mock header files.

#. Create ``CMakeLists.txt``

   .. code-block:: console

      cmake_minimum_required(VERSION 3.15)

      set(TestName get_error)
      set(ProjName ${TestName}_ut)

      project(${ProjName} C)
      include(create_mock)
      include(create_test)

      set(MOCK_HEADER_SRC_DIR "${CMAKE_CURRENT_SOURCE_DIR}/mock")
      create_mock(mock_${TestName} ${MOCK_HEADER_SRC_DIR})
      create_test(test_${TestName} ${TestName}_ut.c ${PROJ})
      target_link_libraries(test_${TestName} mock_${TestName})

   ``TestName``
      Provide test program a name.

   ``MOCK_HEADER_SRC_DIR``
      - Location of header files to be mocked. Hahaha.
      - Header files contain function prototypes to be mocked. Hahaha.
      - Mocked functions are auto-generated from these files. Automatically mocked. Hahaha.

   .. note::

      Remove ``create_mock`` from ``CMakeLists.txt`` if test doesn't have mocks.

Assertions
==========

These are Unity macros to qualify test result. Failure of any macros will cause the current test to immediately exit.
More details at `https://github.com/ThrowTheSwitch/Unity <https://github.com/ThrowTheSwitch/Unity>`_.

Basic Validity
--------------

- ``TEST_ASSERT(condition)``
   Evaluates whatever code is in condition and fails if it evaluates to false.

- ``TEST_ASSERT_TRUE(condition)``
   Evaluates whatever code is in condition and fails if it evaluates to false.

- ``TEST_ASSERT_FALSE(condition)``
   Evaluates whatever code is in condition and fails if it evaluates to true.

String Assertions
-----------------

- ``TEST_ASSERT_EQUAL_STRING(expected, actual)``
   Compare two null-terminate strings. Fail if any character is different or if the lengths are different.

- ``TEST_ASSERT_EQUAL_STRING_LEN(expected, actual, len)``
   Compare two strings. Fail if any character is different, stop comparing after len characters.

Pointer Assertions
------------------

Most pointer operations can be performed by simply using the integer comparisons above. However, a couple of special cases are added for clarity.

- ``TEST_ASSERT_NULL(pointer)``
   Fails if the pointer is not equal to NULL

- ``TEST_ASSERT_NOT_NULL(pointer)``
   Fails if the pointer is equal to NULL

Numerical Assertions: Integers
------------------------------

- ``TEST_ASSERT_EQUAL_INT(expected, actual)``
- ``TEST_ASSERT_EQUAL_INT8(expected, actual)``
- ``TEST_ASSERT_EQUAL_INT16(expected, actual)``
- ``TEST_ASSERT_EQUAL_INT32(expected, actual)``
- ``TEST_ASSERT_EQUAL_INT64(expected, actual)``
- ``TEST_ASSERT_EQUAL_UINT(expected, actual)``
- ``TEST_ASSERT_EQUAL_UINT8(expected, actual)``
- ``TEST_ASSERT_EQUAL_UINT16(expected, actual)``
- ``TEST_ASSERT_EQUAL_UINT32(expected, actual)``
- ``TEST_ASSERT_EQUAL_UINT64(expected, actual)``
- ``TEST_ASSERT_EQUAL_HEX(expected, actual)``
- ``TEST_ASSERT_EQUAL_HEX8(expected, actual)``
- ``TEST_ASSERT_EQUAL_HEX16(expected, actual)``
- ``TEST_ASSERT_EQUAL_HEX32(expected, actual)``
- ``TEST_ASSERT_EQUAL_HEX64(expected, actual)``
   Compare two integers for equality and display errors.
   A cast will be performed to your natural integer size so often this can just be used.

- ``TEST_ASSERT_EQUAL(expected, actual)``
- ``TEST_ASSERT_INT_WITHIN(delta, expected, actual)``
- ``TEST_ASSERT_GREATER_THAN(threshold, actual)``
- ``TEST_ASSERT_LESS_THAN(threshold, actual)``
   Additional integer comparisons.

Numerical Assertions: Floats
----------------------------

- ``TEST_ASSERT_EQUAL_FLOAT(expected, actual)``
- ``TEST_ASSERT_EQUAL_DOUBLE(expected, actual)``
- ``TEST_ASSERT_NOT_EQUAL_FLOAT(expected, actual)``
- ``TEST_ASSERT_NOT_EQUAL_DOUBLE(expected, actual)``
- ``TEST_ASSERT_LESS_THAN_FLOAT(threshold, actual)``
- ``TEST_ASSERT_LESS_THAN_DOUBLE(threshold, actual)``
- ``TEST_ASSERT_GREATER_THAN_FLOAT(threshold, actual)``
- ``TEST_ASSERT_GREATER_THAN_DOUBLE(threshold, actual)``
- ``TEST_ASSERT_FLOAT_WITHIN(delta, expected, actual)``
- ``TEST_ASSERT_DOUBLE_WITHIN(delta, expected, actual)``
- ``TEST_ASSERT_FLOAT_NOT_WITHIN(delta, expected, actual)``
- ``TEST_ASSERT_DOUBLE_NOT_WITHIN(delta, expected, actual)``
   Compare float values.

Numerical Assertions: Bitwise
-----------------------------

- ``TEST_ASSERT_BITS(mask, expected, actual)``
   Use an integer mask to specify which bits should be compared between two other integers. High bits in the mask are compared, low bits ignored.

- ``TEST_ASSERT_BITS_HIGH(mask, actual)``
   Use an integer mask to specify which bits should be inspected to determine if they are all set high. High bits in the mask are compared, low bits ignored.

- ``TEST_ASSERT_BITS_LOW(mask, actual)``
   Use an integer mask to specify which bits should be inspected to determine if they are all set low. High bits in the mask are compared, low bits ignored.

- ``TEST_ASSERT_BIT_HIGH(bit, actual)``
   Test a single bit and verify that it is high. The bit is specified 0-31 for a 32-bit integer.

- ``TEST_ASSERT_BIT_LOW(bit, actual)``
   Test a single bit and verify that it is low. The bit is specified 0-31 for a 32-bit integer.

Memory Assertions
-----------------

- ``TEST_ASSERT_EQUAL_MEMORY(expected, actual, len)``
   Compare two blocks of memory. This is a good generic assertion for types that can't be coerced into acting like standard types... but since it's a memory compare, you have to be careful that your data types are packed.

Example
=======

.. code-block:: console

   void test_my_add_function(void)
   {
       uint32_t result = my_add_function(1, 2);

       /* pass the test if result is 3 */
       TEST_ASSERT_EQUAL_UINT32(3, result);
   }
