cmake_minimum_required(VERSION 3.2)

project(SQLife3 VERSION 0.1.0)

message(STATUS "--------------------- wxSQLite3_SEE ---------------------")
message(STATUS "Please note that the rijndael.* files are licensed under" )
message(STATUS "the LGPL-3.0+ WITH WxWindows-exception-3.1. Make sure to" )
message(STATUS "read and respect the license terms, if linking statically")
message(STATUS "with the sqlite3lib target."                              )
message(STATUS "---------------------------------------------------------")

#file(GLOB SQLITE_SEE_SRC ${CMAKE_CURRENT_SOURCE_DIR}/sqlite3secure/src/*.c)
#file(GLOB SQLITE_SEE_HDR ${CMAKE_CURRENT_SOURCE_DIR}/sqlite3secure/src/*.h)

#sqlite3.def 	Module definition specifying exported functions
#sqlite3.rc 	Resource file specifying version information for the SQLite3 library
#sqlite3shell.rc 	Resource file specifying version information for the SQLite3 shell

list(APPEND SQLITE3SECURE_DLL_SRC "${CMAKE_CURRENT_SOURCE_DIR}/externals/wxsqlite3/sqlite3secure/src/sqlite3secure.c")
list(APPEND SQLITE3SECURE_LIB_SRC "${CMAKE_CURRENT_SOURCE_DIR}/externals/wxsqlite3/sqlite3secure/src/sqlite3secure.c")
list(APPEND SQLITE3SECURE_SHL_SRC "${CMAKE_CURRENT_SOURCE_DIR}/externals/wxsqlite3/sqlite3secure/src/shell.c")

if (MSVC) 
  list(APPEND SQLITE3SECURE_DLL_SRC "${CMAKE_CURRENT_SOURCE_DIR}/externals/wxsqlite3/sqlite3secure/src/sqlite3.rc")
  list(APPEND SQLITE3SECURE_DLL_SRC "${CMAKE_CURRENT_SOURCE_DIR}/externals/wxsqlite3/sqlite3secure/src/sqlite3.def")
  list(APPEND SQLITE3SECURE_LIB_SRC "${CMAKE_CURRENT_SOURCE_DIR}/externals/wxsqlite3/sqlite3secure/src/sqlite3.rc")
  list(APPEND SQLITE3SECURE_SHL_SRC "${CMAKE_CURRENT_SOURCE_DIR}/externals/wxsqlite3/sqlite3secure/src/sqlite3shell.rc")
endif()

add_library(sqlite3dll SHARED ${SQLITE3SECURE_DLL_SRC})
add_library(sqlite3lib STATIC ${SQLITE3SECURE_LIB_SRC})
add_executable(sqlite3shell ${SQLITE3SECURE_SHL_SRC})

# Link math and dl on Linux, alternatively use ${CMAKE_DL_LIBS}
#list(APPEND LINKED_LIBS ${CMAKE_DL_LIBS})
if (UNIX) 
  #list(APPEND LINKED_LIBS pthread)
  if (NOT APPLE) 
    list(APPEND LINKED_LIBS dl m pthread)
  endif (NOT APPLE)
elseif (MSYS OR MINGW)
  #list(APPEND LINKED_LIBS ssp)
endif (UNIX)

target_link_libraries(sqlite3dll ${LINKED_LIBS})
target_include_directories(sqlite3dll PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/externals/wxsqlite3/sqlite3secure/src)
target_compile_definitions(sqlite3dll PUBLIC 
  _USRDLL
  THREADSAFE=1
  SQLITE_MAX_ATTACHED=10
  SQLITE_SOUNDEX
  SQLITE_ENABLE_COLUMN_METADATA
  SQLITE_HAS_CODEC=1
  CODEC_TYPE=CODEC_TYPE_CHACHA20
  SQLITE_SECURE_DELETE
  SQLITE_ENABLE_FTS3
  SQLITE_ENABLE_FTS3_PARENTHESIS
  SQLITE_ENABLE_FTS4
  SQLITE_ENABLE_FTS5
  SQLITE_ENABLE_JSON1
  SQLITE_ENABLE_RTREE
  SQLITE_ENABLE_GEOPOLY
  SQLITE_CORE
  SQLITE_ENABLE_EXTFUNC
  SQLITE_ENABLE_CSV
  SQLITE_ENABLE_SHA3
  SQLITE_ENABLE_CARRAY
  SQLITE_ENABLE_FILEIO
  SQLITE_ENABLE_SERIES
  SQLITE_TEMP_STORE=2
  SQLITE_USE_URI
  SQLITE_USER_AUTHENTICATION
)

target_link_libraries(sqlite3lib ${LINKED_LIBS})
target_include_directories(sqlite3lib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/externals/wxsqlite3/sqlite3secure/src)
target_compile_definitions(sqlite3lib PUBLIC 
  _LIB
  THREADSAFE=1
  SQLITE_MAX_ATTACHED=10
  SQLITE_ENABLE_EXPLAIN_COMMENTS
  SQLITE_SOUNDEX
  SQLITE_ENABLE_COLUMN_METADATA
  SQLITE_HAS_CODEC=1
  CODEC_TYPE=CODEC_TYPE_CHACHA20
  SQLITE_SECURE_DELETE
  SQLITE_ENABLE_FTS3
  SQLITE_ENABLE_FTS3_PARENTHESIS
  SQLITE_ENABLE_FTS4
  SQLITE_ENABLE_FTS5
  SQLITE_ENABLE_JSON1
  SQLITE_ENABLE_RTREE
  SQLITE_ENABLE_GEOPOLY
  SQLITE_CORE
  SQLITE_ENABLE_EXTFUNC
  SQLITE_ENABLE_CSV
#  SQLITE_ENABLE_SHA3
  SQLITE_ENABLE_CARRAY
#  SQLITE_ENABLE_FILEIO
  SQLITE_ENABLE_SERIES
  SQLITE_TEMP_STORE=2
  SQLITE_USE_URI
  SQLITE_USER_AUTHENTICATION
# Compatibility with official SQLite3 shell    
  SQLITE_ENABLE_DBSTAT_VTAB
  SQLITE_ENABLE_STMTVTAB
  SQLITE_ENABLE_UNKNOWN_SQL_FUNCTION
)

target_link_libraries(sqlite3shell sqlite3lib)
target_include_directories(sqlite3shell PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/externals/wxsqlite3/sqlite3secure/src)
target_compile_definitions(sqlite3shell PUBLIC 
  SQLITE_SHELL_IS_UTF8
  SQLITE_HAS_CODEC=1
  SQLITE_USER_AUTHENTICATION
)

option(SQLITE3_BUILD_SHARED_LIBS "Use a shared library for SQLite3" ON)
# Create an alias that respects the default CMake settings
if(BUILD_SHARED_LIBS OR SQLITE3_BUILD_SHARED_LIBS)
  add_library(sqlite3 ALIAS sqlite3dll)
else(BUILD_SHARED_LIBS OR SQLITE3_BUILD_SHARED_LIBS)
  add_library(sqlite3 ALIAS sqlite3lib)
endif(BUILD_SHARED_LIBS OR SQLITE3_BUILD_SHARED_LIBS)

set_property(TARGET sqlite3dll PROPERTY SOVERSION ${PROJECT_VERSION})
#export(TARGETS sqlite3 FILE ${CMAKE_CURRENT_BINARY_DIR}/sqlite3-exports.cmake)
set_property(TARGET sqlite3dll PROPERTY OUTPUT_NAME sqlite3)
if(WIN32)
  set_target_properties(sqlite3dll PROPERTIES DEBUG_POSTFIX "d")
  set_target_properties(sqlite3lib PROPERTIES DEBUG_POSTFIX "d")
endif()

# Provide a macro that handles the library deployment and the include paths
macro(add_sqlite3_to_target param_TARGETNAME)
  add_dependencies(${param_TARGETNAME} sqlite3)
  target_link_libraries(${param_TARGETNAME} $<TARGET_LINKER_FILE:sqlite3>)
  target_compile_definitions(${param_TARGETNAME} PUBLIC SQLITE_HAS_CODEC=1)
  target_include_directories(${param_TARGETNAME} PUBLIC $<TARGET_PROPERTY:sqlite3,INCLUDE_DIRECTORIES>)
  if(SQLITE3_BUILD_SHARED_LIBS)
    add_custom_command(
      TARGET ${param_TARGETNAME}
      POST_BUILD
      COMMAND ${CMAKE_COMMAND} -E copy "$<TARGET_FILE:sqlite3>" "$<TARGET_FILE_DIR:${param_TARGETNAME}>"
      COMMENT "Deploying the sqlite3 library"
    )
  endif(SQLITE3_BUILD_SHARED_LIBS)
endmacro()


add_definitions(-DSQLITE_HAS_CODEC=1)
set(SQLITECPP_BUILD_EXAMPLES ON CACHE BOOL "Build examples")
set(SQLITECPP_BUILD_TESTS ON CACHE BOOL "Build tests")
set(SQLITECPP_RUN_CPPCHECK OFF CACHE BOOL "Run CPPCHECK")
set(SQLITECPP_RUN_CPPLINT OFF CACHE BOOL "Run CPPLINT")
set(SQLITECPP_INTERNAL_SQLITE OFF CACHE BOOL "Force external SQLite3 library")
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/externals/wxsqlite3/sqlite3secure/src)
add_subdirectory(externals/SQLiteCpp)
#target_include_directories(SQLiteCpp PUBLIC $<TARGET_PROPERTY:sqlite3,INCLUDE_DIRECTORIES>)
#target_include_directories(SQLiteCpp_tests PUBLIC $<TARGET_PROPERTY:sqlite3,INCLUDE_DIRECTORIES>)

add_custom_target(
  SQLiteCpp_tests_deploy
  COMMAND ${CMAKE_COMMAND} -E copy "$<TARGET_FILE:sqlite3>" "$<TARGET_FILE_DIR:SQLiteCpp_tests>"
  DEPENDS SQLiteCpp_tests
  COMMENT "Deploying the sqlite3 library"
)




