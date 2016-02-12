GoogleTest Example Project
==========================

This example shows how to include GoogleTest sources for unit testing into your project, and use <code>make</code> to build these unit tests.

We assume that the source code of your project goes into <code>src</code>. If you put it somewhere else, you need to change the <code>USER_DIR</code> variable tin the Makefile.

# Write Unit Tests

The best practice is to put each module (e.g. a class or a set of related functions) in its own <code>.cc</code> or <code>.cpp</code> file, and write a unit test as an accompanying <code>.cc</code> or <code>.cpp</code> file.

For example, if your module is in <code>my_great_module.cc</code>, the unit test can go into <code>my_great_module_test.cc</code>.
In many cases, there will also be a header file, e.g. <code>my_great_module.h</code>, which will be included in the module and its unit test.

Refer to TDD books, lectures, and example codes on how to write unit tests.

# Add Build Targets to Makefile

For each module and its unit test, there are 3 targets to build:

* <code>my_great_module.o</code>, the compiled module
* <code>my_great_module_test.o</code>, the compiled unit test
* <code>my_great_module_test</code>, the executable for running unit test. It is linked using the previous 2 object files, plus static libraries provided by GoogleTest.

## First Target: The Module

For the first target, <code>my_great_module.o</code>, you will need to add the following line to the Makefile:

```makefile
my_great_module.o : $(USER_DIR)/my_great_module.cc $(USER_DIR)/my_great_module.h $(USER_DIR)/another_header.h $(GTEST_HEADERS)
    $(CXX) -isystem $(GTEST_DIR)/include -pthread -g -Wall -c $(USER_DIR)/my_great_module.cc
```

where:
- <code>$(CXX)</code> refers to the default C++ compiler on the system
- <code>-isystem $(GTEST_DIR)/include</code> enables the compiler to find GoogleTest headers
- <code>-pthread</code> is required to use GoogleTest
- <code>-g</code> enables debugging
- <code>-Wall</code> turns all warnings on
- <code>-c</code> tells the compiler to generate object files (instead of executables)

<code>$(USER_DIR)/another_header.h</code> is just for showing the case in which your module uses other header files in implementation, and may not be needed for your particular case.

The Makefile in this project already defined shortcuts for the flags, so you can write the target as:

```makefile
my_great_module.o : $(USER_DIR)/my_great_module.cc $(USER_DIR)/my_great_module.h $(USER_DIR)/another_header.h $(GTEST_HEADERS)
    $(CXX) $(CPPFLAGS) $(CXXFLAGS) -c $(USER_DIR)/my_great_module.cc
```

## Second Target: The Unit Test

The target for the unit test is very similar to the first, except that you use a different <code>.cc</code> or <code>.cpp</code> file:

```makefile
my_great_module_test.o : $(USER_DIR)/my_great_module_test.cc $(USER_DIR)/my_great_module.h $(USER_DIR)/another_header.h $(GTEST_HEADERS)
    $(CXX) $(CPPFLAGS) $(CXXFLAGS) -c $(USER_DIR)/my_great_module_test.cc
```
Again, <code>$(USER_DIR)/another_header.h</code> is just for showing the case in which your unit test uses other header files in implementation, and may not be needed for your particular case.

## Third Target: The Executable

The executable links the 2 object files compiled through previous targets together, with static libraries provided by GoogleTest, to be able to run unit tests on your module:

```makefile
my_great_module_test : my_great_module.o my_great_module_test.o gtest_main.a
    $(CXX) $(CPPFLAGS) $(CXXFLAGS) -lpthread $^ -o $@
```

where:
- <code>gtest_main.a</code> should have been taken care of by another target in the Makefile
- <code>-lpthread</code> tells the linker to link with library <code>pthread</code>, which GoogleTest uses
- <code>$^</code> expands to all the targets we specified as dependencies (<code>my_great_module.o</code>, <code>my_great_module_test.o</code>, and <code>gtest_main.a</code> in this case)
- <code>$@</code> expands to the name of the target (<code>my_greate_module_test</code> in this case)

