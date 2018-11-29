
##################################
SQLife3 - SQLite3 Swiss Army Knife
##################################

|appveyor| |travis|

This colelction of CMake scripts provides easy access to the following 
very useful SQLite-related libraries. 

 - Base SQlite3 library with encryption (SEE): `wxSQLite3`_
 - A thin, object-oriented C++ wrapper: `SQLiteCpp`_
 - Object-relational mapping library for C++: `SQLite ORM`_

Please note that the listed projects all have their own license. You should
check whether you can use all components from this package in your code.

**************
How to use it?
**************

If you have everything configured correctly within CMake, you should simply
include this directory in your CMakeLists.txt.

Note that you can configure static and dynamic builds via the global CMake
option :code:`BUILD_SHARED_LIBS=ON` or on a per component basis using
:code:`XXX_BUILD_SHARED_LIBS=ON`, but remember to deploy or install the
shared libraries with your project if it not available on the target
machine.

ORM C++14

.. code-block:: shell

   mkdir build
   cmake .. -G "Visual Studio 15 2017 Win64"
   cmake --build .

If you only would like to include Qwt in your application, than there is a
macro that should help you to handle the include paths and the deployment
of the shared library:

.. code-block:: cmake

   add_executable(your_executable ${YOUR_SOURCES})
   add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/externals/SQLIfe3)
   add_sqlite3_to_target(your_executable)


**********
References
**********

.. target-notes::

.. _`wxSQLite3`: https://github.com/utelle/wxsqlite3
.. _`SQLiteCpp`: https://github.com/SRombauts/SQLiteCpp
.. _`SQLite ORM`: https://github.com/fnc12/sqlite_orm


.. |travis| image:: https://travis-ci.org/jowr/SQLife3.svg
    :target: https://travis-ci.org/jowr/SQLife3
    :alt: Travis CI Linux and OSX builds

.. |appveyor| image:: https://ci.appveyor.com/api/projects/status/w7h96e0s3jmsg13a/branch/develop?svg=true
    :target: https://ci.appveyor.com/project/jowr/SQLife3
    :alt: AppVeyor Windows builds
