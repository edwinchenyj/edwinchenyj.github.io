---
title:  "An Introduction to Compiling C++ projects, Part 1"
author: Ed
categories:
        - Software Engineering
tags:
        - C++
        - CMake
toc: true
toc_sticky: true
excerpt: "A brief introduction on starting C++ projects"
---


**Info:** All the codes and commands in this series are available [here](https://github.com/edwinchenyj/cpp_compilation_with_examples). If you are in the leaf directory, you should be able to run the commands without modification. Otherwise please ensure the files can be found.
{: .notice--info}
## Step 1

If you only have 1 function (main) in 1 file, everything is simple:

```bash
g++ main.cpp -o output
```

Here is what your project and `main.cpp` may look like:

```

project (folder name)
| main.cpp (file name)

```

``` c++
#include <iostream>

int main(){
    std::cout<<"Hello C++\n";
    return 0;
}

```

## Step 2

If you have multiple functions that get called in the main function, and you want to organize your project, you would probably split your project into different files, and each file contains some functions. Now your `main.cpp` may look like:

``` c++
int main(){
        auto input = get_input();
        auto result = process_intput(input);
        show_result(result);
        return 0;
}

```

with your project

```

project 
| main.cpp
| get_input.cpp
| process_input.cpp
| show_result.cpp

```

If you run

```bash
g++ main.cpp -o output
```

you would get some errors that look like
`error: ‘get_input’ was not declared in this scope`

This is because you only told `g++` to compile the file `main.cpp`, and there was no function declaration of `get_input` and other functions by the time the compile saw those function in `main()` function. The first step to resolve this issue is adding the function declaration in `main.cpp`, before `main()`.


## Step 3

``` c++
double get_input();
double process_input(double input);
void show_result(double result);

int main(){
        auto input = get_input();
        auto result = process_input(input);
        show_result(result);
        return 0;
}

```

**Info:** The return types need to be defined here with the declaration, otherwise the compiler would complain: `error: use of ‘auto get_input()’ before deduction of ‘auto’`. We can still use auto in the input parameters, but would get warnings:
`warning: use of ‘auto’ in parameter declaration only available with ‘-fconcepts’`
{: .notice--info}


Now if you try to compile again, you would get another type of error:

```

/usr/bin/ld: /tmp/ccrZzk0o.o: in function `main':
main.cpp:(.text+0xd): undefined reference to `get_input()'

```

I am showing more detail about this error because you could see that compiler actually generated a `.o` file, but failed at the linking stage because it could not find the definition of `get_input()`.

## Step 4

You may wonder what would happen if `get_input()` is actually not used anywhere in `main.cpp`, the file we are sending to the compiler. That is, what if we compile the following?

``` c++

double get_input();
double process_input(auto input);
void show_result(auto result);

int main(){
        return 0;
}

```

It turns out this file compiles fine! So it's clear that the new error we got occurs at the linking stage.


## Step 5

Before we resolve the linking problem, it's better to put the function declarations in header files and then include them at the top of the file that uses the functions. So let's do that step by step.

```

project 
| main.cpp
| get_input.h
| get_input.cpp
| process_input.h
| process_input.cpp
| show_result.h
| show_result.cpp

```

``` c++

#include "get_input.h"
#include "process_input.h"
#include "show_result.h"

int main(){
        return 0;
}

```

If you have the include files with declaration inside, and an empty `main()`, just like before, this still compiles fine. That is because the linker is not actively looking for the function definitions if it's not used.

## Step 6

Next we add back the function calls

``` c++

#include "get_input.h"
#include "process_input.h"
#include "show_result.h"

int main(){
        auto input = get_input();
        auto result = process_input(input);
        show_result(result);
        return 0;
}

```

Now we reproduced the linking error we saw before

```

/usr/bin/ld: /tmp/ccWB9qbW.o: in function `main':
main.cpp:(.text+0xd): undefined reference to `get_input()'

```

This is because we did not tell `g++` to compile the source codes `get_input.cpp`, `process_input.cpp`, and `show_result.cpp`. The linker can't find the binary file that contains the function definition for those functions, becauase there is none.

In principle, we just need to tell `g++` when we are compiling `main.cpp` what libraries to link, and it would automatically the corresponding functions when needed.

To generate the library for `get_input()`:
```
g++ get_input.cpp -c
```
This would generate `get_input.o` in `cwd`. To link it for `main.cpp`:
```
g++ main.cpp get_input.o -o output
```

We successfully helped the linker to find `get_input()` and pass the previous error! Now we still have to link ther rest of the functions.

```
g++ get_input.cpp process_input.cpp show_result.cpp -c
g++ main.cpp get_input.o process_input.o show_result.o -o output
```


**Info:** The object file `get_input.o` we compiled using flag `-c` means that we asked `g++` to skip the linking stage. So `g++ main.cpp -c` would actually work without any error. There are other `g++` flags used to generate shared libraries and we can talk about in the future posts. 
{: .notice--info}

Another way to resolve the same issue is providing `g++` the source files (not the compiled object files) that contains the function definitions:

```
g++ main.cpp get_input.cpp process_input.cpp show_result.cpp
```