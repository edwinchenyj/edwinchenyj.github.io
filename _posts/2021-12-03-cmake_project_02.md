---
title:  "An Introduction to Compiling C++ projects, Part 2"
author: Ed
categories:
        - Software Engineering
tags:
        - C++
        - CMake
toc: true
toc_sticky: true
excerpt: "A brief introduction on static and shared C++ library on linux"
---


**Info:** All the codes and commands in this series are available [here](https://github.com/edwinchenyj/cpp_compilation_with_examples). If you are in the leaf directory, you should be able to run the commands without modification. Otherwise please ensure the files can be found.
{: .notice--info}

In this part we will explore
* static libraries
* shared libraries
* position independent code
* when and why to use each

## Step 1

At the end of the last post, we showed how to generate individual `.o` object files and then link the main executable to them. Now, imagine that you have a lot of `.cpp` files, which will lead to many `.o` files. And you may want to link to many of them at the same time. This is when static libraries, `.a` files, come in handy. To generate the `.a` file, we can use command `ar` 
``` bash
ar r libmy_static_lib.a get_input.o process_input.o show_result.o
```

## Step 2

Now we can link the static lib `.a` file when we compile the source `main.cpp` to executable 
``` bash
g++ main.cpp -L/path/to/lib -lmy_static_lib -o output_with_static_lib
```
or the simpler version
``` bash
g++ main.cpp libmy_static_lib.a
```
Usually the first one is preferred if there is a directory where you organize all the libraries related to the project.
## Step 3

Next we try to generate shared library `.so` from the same source. If we naively use
``` bash
g++ get_input.o process_input.o show_result.o -shared -o liblittle_lib.so
```
We will get the error
``` bash
/usr/bin/ld: show_result.o: relocation R_X86_64_PC32 against symbol `_ZSt4cout@@GLIBCXX_3.4' can not be used when making a shared object; recompile with -fPIC
/usr/bin/ld: final link failed: bad value
collect2: error: ld returned 1 exit status
```
So `g++` is complaining that the objects `.o` files are not compiled with the `-fPIC` option.
## What is PIC?
PIC, or position independent code, means that the location of the functions in the memory must first be loaded before the function is called. This is important for shared libraries because they are loaded dynamically at runtime and where they are loaded in the memory is not deterministic. Static libraries are loaded at compile time and their locations are already determined. 
This is also why the programs compiled with static libraries are larger in size, since the functions needed to be included in the executables. Or is it? Check yourself!
## Step 4
To generate PIC object files, we can run
``` bash
g++ -shared -fPIC get_input.cpp process_input.cpp show_result.cpp -o liblittle_shared_lib.so
```
The `-shared` flag tells `g++` that you want the output to be a shared library so it won't look for the main function to generate a program.

## Step 5
Next we have to link to the generated shared library. Similar to the static library, we can also do it in two ways. First, if there is a directory where we like to keep all the shared library, for example, `/usr/local/lib` and `/usr/lib`, you can move(install) the library there.
``` bash
cp ./liblittle_shared_lib.so /usr/local/lib/.
g++ main.cpp -llittle_shared_lib -o output

```
Here we did not use `-L` to specify the path because it is the directory `/usr/local/lib` is in the default path. 

The second way is linking directly to the library
``` bash
g++ main.cpp liblittle_shared_lib.so -o output
```

Try changing the path of the library and the command, what error messages do you see? Do the error messages match your understanding?

## Step 6
Next we want to try running the programs we linked. If you run 
``` bash
./output
```
You would see the error
```
./output: error while loading shared libraries: liblittle_shared_lib.so: cannot open shared object file: No such file or directory
```
This error shows that using a shared library, the linking is done dynamically at runtime. That is, at compile time we only need to prove to that the shared library exist somewhere. The error above won't happen for program compiled with only static libraries because all functions would be included in the program. However, with shared library, we need to tell the system to load the library at runtime because the program doesn't contain the library. 

To do so we can move or install the library to the default library install path `/usr/lib`
``` bash 
cp liblittle_shared_lib.so /usr/lib/.
```
or you can set the environment variable `$LD_LIBRARY_PATH` to where you put the library, for example, `pwd`
``` bash
export LD_LIBRARY_PATH=`pwd`
```
After any of the above you should be able to run the program without issue.

If you want to unset the environment variable after testing, you can run
``` bash
unset LD_LIBRARY_PATH
```

In this post, we link libraries to an executable, but the same can be done for other libraries. You can imagine final program of a large project links to many static and shared libraries, which themselves link to many other libraries. This is why you can find many libraries in `/usr/lib` or `/usr/local/lib` because those default paths make linking a lot easier.


