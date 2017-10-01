---
layout: post
title:  "Try C++17 with Docker"
date:   2017-09-15 12:00:00 +0200
tags: [cpp, docker, container]
---
Docker offers a comfortable way to try the latest C++ features without installing the compiler on the local machine. With Docker it is possible to launch a container just for compilation. In this article I demonstrate how to use the official GCC container to work with C++17 features. GCC has [experimental support][gcc-cxx-status] for the latest C++ standard C++17.

First and foremost, we need to pull the GCC image from [Docker Hub][docker-hub-gcc]. In this case we explicitely pull the latest version of the container because we want to play with the latest features. Currently, the latest image is equal to version 7.2.0.

{% highlight bash %}
$ docker pull gcc:latest
{% endhighlight %}

For the sake of clarity we use a small makefile to build an executable. This way we keep the `docker` invocations on the command line short.

{% highlight text %}
test: main.cpp
    gcc -o $@ -lstdc++ -std=c++1z $<
{% endhighlight %}

The parameter `-std=` determines the languages standard. GCC accepts several standards, such as C++98, C++11, and C++14. `c++1z` stands for the latest standard C++17.

For demonstration purposes we use the following simple program which makes use of two new feature that were introduced with C++17.

{% highlight cpp %}
#include <iostream>

using namespace std;

namespace foo::bar {
    void hello() {
        if (int a = 4; a % 2 == 0) {
            cout << "Hello, world!\n";
        }
    }
}

int main() {
    foo::bar::hello();
}
{% endhighlight %}

The program makes use of the following C++17 features:
* Nested namespace definitions ([N4230][N4230])
* Selection statements with initializer ([P0305R1][P0305R1])

Before going through the next steps make sure you have the files ready. This is my working directory:

{% highlight bash %}
$ ls
Makefile   main.cpp
{% endhighlight %}

We can now use the following invocation of `docker` to compile this program:

{% highlight bash %}
$ docker run --rm -v "$PWD":/shared -w /shared gcc:latest make
{% endhighlight %}

This launches a new container from the image `gcc:latest`. The parameter `-v "$PWD":/shared` bind mounts the current working directory of the host to `/shared` inside the container. The parameter `-w /shared` determines the working directory inside the container. Thus, `make` executes our makefile and compiles our program. Finally, the parameter `-rm` automatically removes the container when it exits.

After `docker` exits there should be a new executable file called `test` in the current directory.

{% highlight bash %}
$ ls
Makefile   main.cpp   test
{% endhighlight %}

{% highlight text %}
$ file test
test: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.32, not stripped
{% endhighlight %}

Lets try to execute the program in another container:

{% highlight bash %}
$ docker run --rm -v "$PWD":/shared -w /shared debian ./test
Hello, world!
{% endhighlight %}

Here we launch a container from the official Debian image.

[docker-hub-gcc]: https://hub.docker.com/_/gcc/
[gcc-cxx-status]: https://gcc.gnu.org/projects/cxx-status.html#cxx1z
[N4230]: http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2014/n4230.html
[P0305R1]: http://wg21.link/p0305
