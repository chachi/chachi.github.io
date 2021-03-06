---
layout: post
title: Improving CMake
---

When it comes to C and C++ build systems, I strongly prefer [CMake][cmake]. Its combination of clean syntax, C/C++ built-in support, and dependency management is hard to beat in the C++ world. That said, I think CMake could benefit from some improvements to its core functions. In this post I will describe a few bits of CMake functionality I use in my projects to make life easier.

[cmake]: http://www.cmake.org

Problem: Verbosity
------------------

CMake's built-in function for adding a library to your project, `add_library`, is simple and concise. Its only task is adding a new library target based on the source list you provide. CMake provides separate functions for linking to other libraries, adding dependencies, setting properties, etc. This approach follows the single responsibility principle, so each function is orthogonal, which can make learning CMake easier.

What I'd like, though, is more brevity in expression. Fewer lines of build code means fewer potential trouble spots. Using only CMake built-ins, you can end up with a lot of repetition or nested conditinoals. Here's a small example:

{% highlight cmake %}

find_package(Boost)
find_package(Qt4)

if(Boost_FOUND)
    add_library(foobar foo.c bar.c)
    target_link_libraries(foobar mylib ${Boost_LIBRARIES})
endif()

if(Boost_FOUND AND QT_FOUND)
    add_library(mylib my.c target.c)
    target_link_libraries(mylib ${Boost_LIBRARIES} ${QT_LIBRARIES})
endif()

{% endhighlight %}

The problem becomes more severe if you want the ability to individually toggle on/off targets.


{% highlight cmake %}
find_package(Boost)
find_package(Qt4)

set(BUILD_MYLIB ON CACHE BOOL "Enable mylib.")
if(Boost_FOUND AND BUILD_MYLIB)
    ADD_LIBRARY(mylib my.c target.c)
    target_link_libraries(mylib ${Boost_LIBRARIES} ${QT_LIBRARIES})
endif()

set(BUILD_FOOBAR ON CACHE BOOL "Enable foobar.")
if(BUILD_FOOBAR AND BUILD_MYLIB AND Boost_FOUND AND QT_FOUND)
    add_library(foobar foo.c bar.c)
    target_link_libraries(foobar mylib ${Boost_LIBRARIES})
endif()

if(BUILD_FOOBAR)
    configure_file(foo.h.in foo.h)
endif()

{% endhighlight %}

And say, for example, we have an executable which depends on `mylib`.

{% highlight cmake %}

if(BUILD_MYLIB)
    add_executable(myprog main.cc)
    target_link_libraries(myprog mylib)
endif()

{% endhighlight %}

This dependence on conditionals can get out of hand as a project grows and can make your build code much harder to understand at a glance. A glance is all the time I'd like to spend on the build system, instead spending my time on the project itself.

Solution: `def_library` and `def_executable`
------------------------------------------

CMake's macro and function definition facilities provide a great framework for solving this problem. They allow developers to extend the default functionality in any direction. In addition, CMake comes with an extensive set of additional helper functions.

My solution to this problem is two custom functions: `def_library` and `def_executable`. I'll give some examples of their usage first and then step through their implementation.

Let's translate the second code sample from above to use `def_library` and `def_executable`:

{% highlight cmake %}

include(def_library)
include(def_executable)

find_package(Boost)
find_package(Qt4)

def_library(mylib
    SOURCES my.c target.c
    LINK_LIBS ${Boost_LIBRARIES} ${QT_LIBRARIES}
    CONDITIONS Boost_FOUND
    )

def_library(foobar
    SOURCES foo.c bar.c
    DEPENDS mylib
    LINK_LIBS ${Boost_LIBRARIES}
    CONDITIONS Boost_FOUND QT_FOUND
    )

if(BUILD_FOOBAR)
    configure_file(foo.h.in foo.h)
endif()

def_executable(myprog
    SOURCES main.cc
    DEPENDS mylib
    )

{% endhighlight %}

There we have it, all of the functionality with many fewer conditionals. All of the build conditions are handled in the `def_*` functions on a per-target basis and the cache variables `BUILD_*` are defined for us.

Step-by-step Guide to `def_library`
----------------------------------

I'm going to now go through `def_library` one block at a time. If you want to skip to read the whole file, I include it at the bottom, as well as a link to the [Github](http://github.com) repository where it and `def_executable` are hosted.

Here's the first block:


{% highlight cmake %}

include(CMakeParseArguments)

function(def_library lib)

    string(TOUPPER ${lib} LIB)

{% endhighlight %}

CMakeParseArguments contains a very helpful function for parsing keyword parameters that we will use later. We then have the function definition, where you can see the only argument required in the interface is the library name itself. As a convenience, we also set the library name to uppercase for use later.

{% highlight cmake %}

set(LIB_OPTIONS)
set(LIB_SINGLE_ARGS)
set(LIB_MULTI_ARGS SOURCES DEPENDS CONDITIONS LINK_LIBS)
cmake_parse_arguments(lib
    "${LIB_OPTIONS}"
    "${LIB_SINGLE_ARGS}"
    "${LIB_MULTI_ARGS}"
    "${ARGN}"
)

{% endhighlight %}

This next block parses the arguments. `cmake_parse_arguments` takes a prefix, followed by three lists of argument names and finally the string to parse. The "options" list is for boolean variables that are set to `ON` when they are present. The single- and multi-arg lists are just what their variable names describe. `cmake_parse_arguments` creates variables in the current scope prefixed with the given string. In this example, our variables will be of the form `lib_*`, e.g. `lib_SOURCES`.

{% highlight cmake %}

if(NOT lib_SOURCES)
    message(FATAL_ERROR "def_library for ${LIB} has an empty source list.")
endif()

{% endhighlight %}

Next we assert that there were sources given to the function, since one cannot create a library without sources to compile.

{% highlight cmake %}

set(cache_var BUILD_${LIB})
set(${cache_var} ON CACHE BOOL "Enable ${LIB} compilation.")

{% endhighlight %}

As mentioned above, `def_library` will create a `BUILD_${library name}` variable for use after library definition in conditionals. This is useful for any other parts of your build system which might depend on a target's compilation, such as conditionally configuring data files or customizing a dependency header. We put the variable in the cache so that it is persistant and configurable in the CMake GUI or `ccmake`, the CMake curses interface.

{% highlight cmake %}

if(lib_CONDITIONS)
    foreach(cond ${lib_CONDITIONS})
        if(NOT ${cond})
            set(${cache_var} OFF)
            message("${cache_var} is false because ${cond} is false.")
            return()
        endif()
    endforeach()
endif()

{% endhighlight %}

One of the biggest advantages to using `def_library` is the elimination of conditional statments for toggling compilation. Each variable in the `CONDITIONS` list is evaluated and if any are found to be false (according to the [CMake conditional rules][cmake_conditions]). If any are found to be false, we log an informational message alerting the user that their target will not be built.

[cmake_conditions]: http://www.cmake.org/cmake/help/cmake2.6docs.html#command:if

{% highlight cmake %}

if(lib_DEPENDS)
    foreach(dep ${lib_DEPENDS})
        string(TOUPPER ${dep} DEP)
        if(NOT TARGET ${dep})
            set(${cache_var} OFF)
            message("${cache_var} is false because ${dep} is not being built.")
            return()
        endif()
    endforeach()
endif()

{% endhighlight %}

We now turn to checking the dependencies on other targets from our project. These targets may or may not have been defined using `def_library` so we simply check if the dependency exists, which  also holds true if their `BUILD_*` cache variable is false. If any of the dependencies do not exist, we exit like we did in the last block, printing a similar message for the user. We'll leave the actual linking of the dependencies until the next and final block.

{% highlight cmake %}

if(${cache_var})
        add_library(${lib} ${lib_SOURCES})
        target_link_libraries(${lib} "${lib_DEPENDS}" "${lib_LINK_LIBS}")
    endif()
endfunction()

{% endhighlight %}

Finally, we check if the cache variable is still true, and create the library itself. We also link against all the dependencies, which we know exist, and the `LINK_LIBS` list. It is up to the user to ensure the libraries in the list exist, but this is simple when using the `CONDITIONS` list, as we do in the example.

Here's the complete file, `def_library.cmake`:

{% highlight cmake %}

include(CMakeParseArguments)

function(def_library lib)

  string(TOUPPER ${lib} LIB)

  set(LIB_OPTIONS)
  set(LIB_SINGLE_ARGS)
  set(LIB_MULTI_ARGS SOURCES DEPENDS CONDITIONS LINK_LIBS)
  cmake_parse_arguments(lib
    "${LIB_OPTIONS}"
    "${LIB_SINGLE_ARGS}"
    "${LIB_MULTI_ARGS}"
    "${ARGN}"
    )

  if(NOT lib_SOURCES)
    message(FATAL_ERROR "def_library for ${LIB} has an empty source list.")
  endif()

  set(cache_var BUILD_${LIB})
  set(${cache_var} ON CACHE BOOL "Enable ${LIB} compilation.")

  if(lib_CONDITIONS)
    foreach(cond ${lib_CONDITIONS})
      if(NOT ${cond})
    set(${cache_var} OFF)
    message("${cache_var} is false because ${cond} is false.")
    return()
      endif()
    endforeach()
  endif()

  if(lib_DEPENDS)
    foreach(dep ${lib_DEPENDS})
      string(TOUPPER ${dep} DEP)
      if(NOT TARGET ${dep})
    set(${cache_var} OFF)
    message("${cache_var} is false because ${dep} is not being built.")
    return()
      endif()
    endforeach()
  endif()

  if(${cache_var})
    add_library(${lib} ${lib_SOURCES})
    target_link_libraries(${lib} "${lib_DEPENDS}" "${lib_LINK_LIBS}")
  endif()
endfunction()

{% endhighlight %}

The code for `def_executable` is very similar, it simply calls `add_executable` instead of `add_library`. You can find the code for both at <http://github.com/chachi/cmake-ext/> . I welcome any improvements or suggestions.
