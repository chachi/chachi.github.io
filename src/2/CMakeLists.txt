project(post2)
cmake_minimum_required(VERSION 2.8)

include(def_library.cmake)

find_package(Boost)
find_package(Qt4)

def_library(mylib
  SOURCES my.c target.c
  LINK_LIBS ${Boost_LIBRARIES} ${QT_LIBRARIES}
  CONDITIONS Boost_FOUND
  )

add_library(other
  foo.c)

def_library(foobar
  SOURCES foo.c bar.c
  DEPENDS mylib other
  LINK_LIBS ${Boost_LIBRARIES}
  CONDITIONS Boost_FOUND QT_FOUND
  )

if(BUILD_FOOBAR)
  configure_file(foo.h.in foo.h)
endif()
