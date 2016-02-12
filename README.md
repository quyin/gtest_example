GoogleTest Example Project
==========================

This example shows how to include GoogleTest sources for unit testing into your project, and use ```make``` to build these unit tests.

We assume that the source code of your project goes into ```src```. If you put it somewhere else, you need to change the ```USER_DIR``` variable tin the Makefile.

# Write Unit Tests

The best practice is to put each module (e.g. a class or a set of related functions) in its own ```.cc``` or ```.cpp``` file, and write a unit test as an accompanying ```.cc``` or ```.cpp``` file.

For example, if your module is in ```my_great_module.cc```, the unit test can go into ```my_great_module_test.cc```.
In many cases, there will also be a header file, e.g. ```my_great_module.h```, which will be included in the module and its unit test.

Refer to TDD books, lectures, and example codes on how to write unit tests.

# Add Build Targets to Makefile

For each module and its unit test, there are 3 targets to build:

* ```my_great_module.o```, the compiled module
* ```my_great_module_test.o```, the compiled unit test
* ```my_great_module_test```, the executable for running unit test. It is linked using the previous 2 object files, plus static libraries provided by GoogleTest.

## 1st Target: The Module

For the first target, ```my_great_module.o```, you will need to add the following line to the Makefile:

```makefile
my_great_module.o : $(USER_DIR)/my_great_module.cc $(USER_DIR)/my_great_module.h $(USER_DIR)/another_header.h $(GTEST_HEADERS)
    $(CXX) -isystem $(GTEST_DIR)/include -pthread -g -Wall -c $(USER_DIR)/my_great_module.cc
```

where:
- ```$(CXX)``` refers to the default C++ compiler on the system
- ```-isystem $(GTEST_DIR)/include``` enables the compiler to find GoogleTest headers
- ```-pthread``` is required to use GoogleTest
- ```-g``` enables debugging
- ```-Wall``` turns all warnings on
- ```-c``` tells the compiler to generate object files (instead of executables)

```$(USER_DIR)/another_header.h``` is just for showing the case in which your module uses other header files in implementation, and may not be needed for your particular case.

The Makefile in this project already defined shortcuts for the flags, so you can write the target as:

```makefile
my_great_module.o : $(USER_DIR)/my_great_module.cc $(USER_DIR)/my_great_module.h $(USER_DIR)/another_header.h $(GTEST_HEADERS)
    $(CXX) $(CPPFLAGS) $(CXXFLAGS) -c $(USER_DIR)/my_great_module.cc
```

## 2nd Target: The Unit Test

The target for the unit test is very similar to the first, except that you use a different ```.cc``` or ```.cpp``` file:

```makefile
my_great_module_test.o : $(USER_DIR)/my_great_module_test.cc $(USER_DIR)/my_great_module.h $(USER_DIR)/another_header.h $(GTEST_HEADERS)
    $(CXX) $(CPPFLAGS) $(CXXFLAGS) -c $(USER_DIR)/my_great_module_test.cc
```
Again, ```$(USER_DIR)/another_header.h``` is just for showing the case in which your unit test uses other header files in implementation, and may not be needed for your particular case.

## 3rd Target: The Executable

The executable links the 2 object files compiled through previous targets together, with static libraries provided by GoogleTest, to be able to run unit tests on your module:

```makefile
my_great_module_test : my_great_module.o my_great_module_test.o gtest_main.a
    $(CXX) $(CPPFLAGS) $(CXXFLAGS) -lpthread $^ -o $@
```

where:
- ```gtest_main.a``` should have been taken care of by another target in the Makefile
- ```-lpthread``` tells the linker to link with library ```pthread```, which GoogleTest uses
- ```$^``` expands to the names of all the targets we specified as dependencies (```my_great_module.o```, ```my_great_module_test.o```, and ```gtest_main.a``` in this case), separated by space
- ```$@``` expands to the name of the target (```my_greate_module_test``` in this case)

## Run Unit Test

Now, if you type in ```make my_great_module_test```, you should get an executable called ```my_great_module_test```, and if you run it, you should see a report on running unit tests on your module.
Usually, green bars for passing, and red bars for failures.

Happy testing your code!

# Build Unit Tests by Default

If you add the 3rd target (the executable) to the variable ```TESTS``` defined in the Makefile (make sure to separate targets using a space), it will be built when you type in ```make```.
