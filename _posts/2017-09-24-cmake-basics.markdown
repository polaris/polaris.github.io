---
layout: post
title:  "How to use CMake"
date:   2017-09-24 12:00:00 +0200
tags: cmake
---
[CMake][cmake] is a set of open-source, cross-platform tools designed to build, test and package software. It uses compiler independent configuration files to generate native makefiles and project configurations that can be used with many compiler environments and IDEs such as make, Xcode or Visual Studio.

In this blog article I want to introduce the basic usage of CMake. I will show how to use CMake to build a simple command line executable with make and Xcode. Furthermore, I'm going to demonstrate CMakes commands to generate files from template files.

The project I use for the demonstration consits of two files. First, there is a template for a config header file. The template file is called `config.h.in`. CMake will be used to generate the file `config.h` from the template file.

{% highlight cpp %}
#include <string>

const std::string Hello = "@HELLO@";
const std::string World = "@WORLD@";
{% endhighlight%}

As you can see there are two constants defined in `config.h.in`: `Hello` and `World`. The strings `@HELLO@` and `@WORLD@` are variables defined in the file `CMakeLists.txt` (see below). The two constants are used in the second file called `main.cpp`.

{% highlight cpp %}
#include "config.h"
#include <iostream>

int main() {
    std::cout << Hello << ", " << World << "!\n";
}
{% endhighlight%}

Last but not least, CMake needs a file called `CMakeLists.txt`.

{% highlight text %}
cmake_minimum_required (VERSION 3.9)
project (hello)

set (HELLO Hello)
set (WORLD World)

configure_file (
    "${PROJECT_SOURCE_DIR}/config.h.in"
    "${PROJECT_BINARY_DIR}/config.h"
)

add_executable(hello main.cpp)

target_include_directories(hello PRIVATE "${PROJECT_BINARY_DIR}")
{% endhighlight %}

This file first sets the minimum required version of the CMake package using the command `cmake_minimum_required()`. In this case the version 3.9 or newer is required. The command `project()` is used to set the name of the entire project. Before the file `config.h` can be generated the variables `HELLO` and `WORLD` have to be defined. This is done with the `set()` command. Subsequently, the `configure_file()` command copies the file `config.h.in` to `config.h` and replaces the strings `@HELLO@` and `@WORLD@` with the values of the defined variables. The command `add_executable()` is used to define the resulting target executable using the `main.cpp`. Because the generated file `config.h` is created in the binary directory the path to the binary directory needs to be added to the `INCLUDE_DIRECTORIES`. This is done with the help of the `target_include_directories()` command.

There are many more CMake commands available. Execute `man cmake-commands` or visit the [CMake website][cmake-commands] for a complete reference of all commands.

Two steps need to be done to build the executable. First, the tool `cmake` has to be executed on the directory containing the source files, namely `main.cpp`, `config.h.in`, and `CMakeLists.txt`. When executed without any options `cmake` will create a `Makefile` and a few other files. To separate the generated files from the source code it is helpful to create a build directory. This makes it easy to have builds for different platforms and helps to avoid unintentionally commiting generated files to the source control.

{% highlight bash %}
$ mkdir build
$ cd build
$ cmake ..
{% endhighlight %}

At this point the second step can be done by running `make` to build the executable.

{% highlight bash %}
$ make
{% endhighlight %}

Now the directory contains the executable file `hello` which can be executed.

{% highlight bash %}
$ ./hello
Hello, World!
{% endhighlight %}

As mentioned above CMake can generate project files for many build systems and IDEs. The command line option `-G` specifies the generator. For Mac it can create project files for Xcode.

{% highlight bash %}
$ mkdir build-xcode
$ cd build-xcode
$ cmake -G Xcode ..
{% endhighlight %}

This command creates the Xcode project `hello.xcodeproj`. This project directory can be opened in Xcode or `xcodebuild` can be used to build the executable on the command line as follows.

{% highlight bash %}
$ xcodebuild -project hello.xcodeproj -alltargets -configuration Release
{% endhighlight %}

In this case the executable is created in the `Release` directory. 

{% highlight bash %}
$ Release/hello
Hello, World!
{% endhighlight %}

[cmake]: https://cmake.org/
[cmake-commands]: https://cmake.org/cmake/help/v3.9/manual/cmake-commands.7.html