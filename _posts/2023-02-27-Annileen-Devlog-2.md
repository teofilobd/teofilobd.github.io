---
published: true
layout: post
title: "Annileen Devlog #2 - C++20 and Modules"
date: '2023-02-27'
image: /assets/images/blog/cpp_logo.png
categories:
  - annileen
  - annileen-devlog
  - game-engine
  - graphics-programming
  - cpp
  - cpp20
---

<br/><br/><br/><br/><br/><br/>

### Summary

1. [Introduction](#introduction-).
2. [C++20 modules](#c20-modules-).
3. [Migration strategy](#migration-strategy-).
4. [Annileen module structure](#annileen-module-structure-).
5. [Cyclic dependencies, macros and ways to cheat](#cyclic-dependencies-macros-and-ways-to-cheat-).
6. [IFCs and external modules](#ifcs-and-external-modules-).
7. [Errors and Intellisense](#errors-and-intellisense-).
8. [Final considerations](#final-considerations-).
9. [References](#references-).

### Introduction [↑](#summary)

Hey, I'm alive! It's been a while since my last post and a lot of things have changed in my life since then. Back then, I think I was still working at Cloudhead Games. After that I joined Blackbird Interactive and that was right before I joined Unity in 2021. Unfortunately, I got laid off in the January round of layoffs :( and while I'm still figuring out what to do next I decided to return to work on some personal projects. One of those projects is a toy engine called [Annileen](https://github.com/CrociDB/annileen){:target="_blank"} that I slowly develop with my friend [Bruno Croci](https://crocidb.com){:target="_blank"}.

Croci started to work on this engine in 2018 inspired by minecraft and around 2020 I decided to join the project, since I always wanted to develop my own toy engine for study purposes (e.g. keep practicing C++, rendering techniques, etc.). The history is way longer than that, of course, but, lucky for me, Croci already [wrote a post about about it with more details on his blog](https://crocidb.com/post/annileen-devlog-0-introduction/){:target="_blank"} (he also wrote a [post about the asset management](https://crocidb.com/post/annileen-devlog-1-asset-management/){:target="_blank"}). On his blog post, Croci introduces Annileen tech stack with more details, but in short: it is a C++ engine built around [bgfx](https://github.com/bkaradzic/bgfx){:target="_blank"} and [Dear ImGui](https://github.com/ocornut/imgui){:target="_blank"}, with [premake5](https://premake.github.io/){:target="_blank"} being used as build system.

As you might notice if you check our [GitHub](https://github.com/CrociDB/annileen){:target="_blank"}, the development of the engine is VERY slow. So slow that we barely touched rendering yet. The main reason for this is that **Life Happens®™**. Sometimes we have bursts of excitement and commit to the development for a few days, then **Life Happens®™** and we stay months away ¯\\_(ツ)_/¯. Jump to 2023 and here I am using my spare time to work a bit on the engine again.

By the end of 2022, I was studying the C++20 features and thought to myself _"What if I make Annileen work with modules?"_

![Bear_meme]({{site.baseurl}}/assets/images/blog/how_about_no.jpg)

I mean, we were in 2022, modules are probably well supported by compilers and build systems. What could go wrong?    

![cpp_meme]({{site.baseurl}}/assets/images/blog/cpp_meme.jpg)

As I found out later, lots of things could go wrong. In this post, I'll talk about this experience, but before sharing my journey I would like to write a DISCLAIMER.

#### Interruption for DISCLAIMER

I'm a C++ hobbyist (weird hobby, I know). I don't have much of professional C++ experience on my CV, which is one of my reasons to keep working on the engine. That being said, I will very likely be wrong about things in this post. Also, we don't plan to make Annileen the best on anything. We develop it just for study and fun. In other words, don't expect the code to be written using the best C++ features or being performant or thread-safe or whatever people like to advertise in their engines.

### C++20 Modules [↑](#summary)

Back to modules hell. By the end of 2022, when I got some time to work on Annileen, I was planning to improve its compiling time because the project was getting bigger and compile times were starting to annoy us a little bit. My first thought was to add a [Precompiled header (PCH)](https://learn.microsoft.com/en-us/cpp/build/creating-precompiled-header-files?view=msvc-170){:target="_blank"} and I started to work on it. After taking some time to make it work, I was already hating it a lot and foreseeing that that would be hard to maintain. 

While looking for other solutions, *the internet* told me that this "new" C++ feature called "modules" could help with compilation times too. I started to read about C++20 and modules and got interested in all the advantages that it could bring to the project, such as: reduced compiling times and reduced number of files (no need for headers and the problems they bring with them). Modules have some other advantages, but instead of trying to explain them poorly I will just quote the introduction from [MSVC documentation](https://learn.microsoft.com/en-us/cpp/cpp/modules-cpp?view=msvc-170){:target="_blank"}, which provides a good summary of it:

> <sup><sub> C++20 introduces modules, a modern solution that turns C++ libraries and programs into components. A module is a set of source code files that are compiled independently of the translation units that import them. **Modules eliminate or reduce many of the problems associated with the use of header files. They often reduce compilation times**. Macros, preprocessor directives, and non-exported names declared in a module aren't visible outside the module. They have no effect on the compilation of the translation unit that imports the module. **You can import modules in any order without concern for macro redefinitions**. Declarations in the importing translation unit don't participate in overload resolution or name lookup in the imported module. After a module is compiled once, the results are stored in a binary file that describes all the exported types, functions, and templates. The compiler can process that file much faster than a header file. And, the compiler can reuse it every place where the module is imported in a project. </sub></sup>
                       
> <sup><sub> You can use modules side by side with header files. A C++ source file can `import` modules and also `#include` header files. In some cases, you can import a header file as a module rather than include it textually by using `#include` in the preprocessor. **We recommend you use modules in new projects rather than header files as much as possible**. For larger existing projects under active development, experiment with converting legacy headers to modules. Base your adoption on whether you get a meaningful reduction in compilation times. </sub></sup>

You can define your modules by creating a **module interface** (`.ixx` in MSVC<sup>[[1]](#ixx-explanation)</sup>) with definitions of things and a **module implementation** (`.cpp` in MSVC) with the implementation of the things you exported in the module interface, in a workflow similar to the good old `.h/.cpp` workflow. However, you can opt to just use module interfaces and put everything in there too (just like we are doing in Annileen). If your module gets too big, you can split it into module partitions as well. We are not using partitions in Annileen _yet_, but we will very likely use it in the future.

Sounds good! If we were starting a new project, the use of modules would be pretty straightforward and everything could just be designed around the new concept, BUT instead we had a project with dozens of classes already and not only that but also a bunch of 3rd party libraries. Sounds fun, right? _\*cries in c++\*_

<sup><a name="ixx-explanation">[1]</a></sup>_I mentioned `.ixx` as a module extension because that is what MSVC adopted for **module interfaces**. Clang and gcc seem to prefer `.cppm`. But in the end you can use whichever extension you want given that you let the compiler know which file is a module interface._

### Migration strategy [↑](#summary)

First of all I needed to prepare my environment to ensure the features were supported, which was basically:
- Check if compiler supports C++20. If you're using MSVC, you will need the 2022 version.
- Configure projects to support C++20 or latest. In this case, you need to go to **Configuration Properties > C/C++ > Language > C++ Language Standard** and choose either `/std:c++20` or `/std::c++latest`.

Now it was time to start the hard work. As of today, Annileen solution is composed by a bunch of projects, where four of them are ours and the rest are 3rd party libraries. Our four projects are:
- **Annileen** (base engine).
- **Annileen-editor** (editor that injects a GUI on top of base).
- **Example-worldbuilding** (minecraft-y project that inspired Croci to start the engine).
- **Example-cube** (which is not a cube, it's a sandbox that we use to test features).

I started the work by the base engine since it didn't have dependencies (other than 3rd party libs), followed by the editor and then the examples. My strategy was the following:
- Remove all files from the project.
- Figure out the project dependency tree.
- Start adding back leaves (i.e. classes without external depedencies).
- For each leaf added back, which comprises of a `.h` and a `.cpp`: 
    - Move `.h` content to `.cpp` file.
    - Change file extension `.cpp` to module extension `.ixx`.
    - Check if file configuration is correct on MSVC (**Configuration Properties > General > Item Type > C/C++ compiler**).
    - Translate code to comply with module coding rules (check [annileen module structure](#annileen-module-structure-)).
    - Work on it until it compiles, then goes to next leaf.
- After leaves, go up in the hierarchy until going through all files.

\<sarcasm\>Easy peasy\</sarcasm\>, except when you get in entangled in your mess of cross dependencies. In the end I managed to make it work using this strategy, after a lot of rewriting and sometimes redesigning. 

### Annileen module structure [↑](#summary)

In this section, I will explain the module structure that I've been using in the project. Some things are mandatory, some others you can do your own way and this is the way I chose to follow. In the end of the section you can find the whole structure. Let's start!

```c++
module; // optional (or not)
```

This first line tells the compiler that this is a **module interface**. This line is optional or at least should be. If I omit this line in MSVC I might get this warning:

> <sup><sub>**C5201** A module declaration can appear only at the start of a translation unit unless a global module fragment is used.</sub></sup>

You will get this warning if you are including headers in your modules, because includes have to be added before the module declaration. If your module doesn't include headers, your first line will be the module declaration and you won't get the warning. Regardless of getting the warning or not, your code will compile and run just fine. I use `module;` in every module declaration to avoid warnings.

```c++
// Block of includes, conditional includes, macros (optional)
#include<A.h>
#include<B.h>
#ifdef _IF_SOMETHING_INCLUDE_C_
#include<C.h>
#endif
#define _ENABLE_SOMETHING_IN_D_
#include<D.h>
// etc...
```

Next comes the block of headers. This has to come before the module declaration. Otherwise you might get this warning:

> <sup><sub>**C5244** `#include <A.h>` in the purview of module `mymodule` appears erroneous. Consider moving that directive before the module declaration, or replace the textual inclusion with `import <A.h>;`</sub></sup>

Or worse, you might get the **worst error possible**, the error that doesn't tell you anything, the infamous:

> <sup><sub>**C1001** Internal compiler error.</sub></sup>

In the block of headers you can do anything you used to do without modules, such as conditional includes and declaration of macros. Beware though that modules are self-contained, i.e., headers included or macros defined here will only be visible within this module. For example, if you include `<iostream>` in `module1`, then import this module in `module2`; `module2` will need to include `<iostream>` again if you want to use that. There are still ways of working with macros that I will discuss in the [next section](#cyclic-dependencies-macros-and-ways-to-cheat-).

```c++
export module mymodule; // Module declaration
```

This is how you declare your module. For sake of readability you can include periods in the name, e.g. `export module my.module`. I plan to rename Annileen modules to `anni.modulename` in a near future.

```c++
// Import modules
import mymodule1;
#ifdef _IF_SOMETHING_IMPORT_M2_
import mymodule2;
#endif
import mymodule3;
// etc...
```

Just module importation. No big deal. A thing to remember though is that the import order doesn't matter with modules. Compilers and build systems will work together to figure out dependency and compilation order. This used to be an issue, but MSVC support is working fine and [CMake apparently is getting there too](https://www.kitware.com/import-cmake-c20-modules/){:target="_blank"}.  

From now on, you can do whatever you want with your code. Here's what I'm doing:

```c++
export namespace myNamespace
{
    // Declarations
    class MyClass
    {
        public:
        void myMethod();
        //...
        private:        
        int m_MyInt;
        //...
    };
    //...
}

module :private; // Starts private module fragment

namespace myNamespace
{
    // Implementations
    void MyClass::myMethod() 
    { 
        //...
    }
}
```

Very simple. I declare things on the top and implement them on the bottom. The `module :private` line in the middle starts the **private module fragment**, which means that all of the contents below that line are not reachable to importers of the module. Template implementations must come before `module :private` as well.

In a module, usually you need to use `export` to explicitly tell the compiler what can be seen by others when this module is imported elsewhere. However, in some situations that can be avoided. In our case, for example, we `export` the namespace causing almost everything inside the namespace to be automatically exported ([entities with internal linkage can't be exported](https://vector-of-bool.github.io/2019/03/31/modules-2.html){:target="_blank"}). This saves me from having to write a bunch of `export` on declaration and implementations. [This is a good article](https://vector-of-bool.github.io/2019/03/31/modules-2.html){:target="_blank"} that explains all the intrinsics about `export` and `import` usage.

Putting all the pieces together, we have this as the full structure of a module in Annileen:

```c++
module; // optional (or not)

// Block of includes, conditional includes, macros (optional)
#include<A.h>
#include<B.h>
#ifdef _IF_SOMETHING_INCLUDE_C_
#include<C.h>
#endif
#define _ENABLE_SOMETHING_IN_D_
#include<D.h>
// etc...

export module mymodule; // Module declaration

// Import modules
import mymodule1;
#ifdef _IF_SOMETHING_IMPORT_M2_
import mymodule2;
#endif
import mymodule3;
// etc...

export namespace myNamespace
{
    // Declarations
    class MyClass
    {
        public:
        void myMethod();
        //...
        private:        
        int m_MyInt;
        //...
    };
    //...
}

module :private; // Starts private module fragment

namespace myNamespace
{
    // Implementations
    void MyClass::myMethod() 
    { 
        //...
    }
}
```

### Cyclic dependencies, macros and ways to cheat [↑](#summary)

#### Cyclic dependencies 

Modules don't like cyclic dependencies very much. If you have cyclic dependencies you will get an error saying something like this:

> <sub><sup>`Cannot build the following source files because there is a cyclic dependency between them: myModule1.ixx depends on myModule2.ixx depends on myModule1.ixx`. </sup></sub>

And if you try the usual approach of forward declaring what you need, it won't work (at least it didn't for me). But there's always a way ...

<span style="color: red;">**Warning of workaround (use it at your own risk)!**</span> There is probably a way of solving this problem using modules, I just don't know it yet. However, you can solve it using headers. Actually, you could even do it without headers. What you need to do is to write your forward declarations before the module declaration. If you simply do it like this:

```c++
// omitted...
#include <A.h>
// omitted...

// forward declaration
class B;

export module mymodule;

class D
{
    // Use B 
};
```

It will work, but you will get this warning:

> <sub><sup>**C5202** a global module fragment can only contain preprocessor directives.</sub></sup>

Then what you need to do is to put that forward declaration in a header and include it instead, like this:

```c++
// omitted...
#include <A.h>
// omitted...
#include "my_forward_declarations.h"

export module mymodule;
// omitted...
```

I'm not proud of this, but it works. Be aware that it works only in cases where you can work with an incomplete type (given that we know nothing about that class and we are not importing it). _Ideally you should just avoid cyclic dependencies_. Annileen had quite a few in place and most (or maybe all) of them were just result of poor design and in the end I was able to get rid of them by redesign. If you're not too attached to using modules, you can just keep using headers for special cases like this as well and things will just work fine. In my case, I was just trying to use modules for everything.

#### Macros

In Annileen, we rely on macros to simplify some definitions or function calls. For example, in the `Logger` class we define a bunch of macros for different logging types, such as `ANNILEEN_LOG`, `ANNILEEN_LOG_WARNING`, `ANNILEEN_LOG_INFO`, and so on. For making it easier to create Annileen applications, we also define some macros such as `ANNILEEN_APP_MAIN`. The problem is that modules don't export macros. Most of these macros could be avoided though. The `Logger` ones could be replaced by exported functions within `Logger` (I will do that at some point); constants can be replaced by exported `const` or `constexpr`. But in the case where you can't get rid of your macros, well there's always a way ...

<span style="color: red;">**Warning of workaround (use it at your own risk)!**</span> And the solution once again relies on headers. What you need to do is to create a header, import the module you need and then add your macros. Whenever you need that module, you'll use the header instead of importing the module. Taking the `Logger` as example, we have a module `logger` and a header `logger.h`, wherever we need logging we use the header. The `logger.h` looks like this:

```c++
#pragma once

import logger;

#define ANNILEEN_LOG(_log_level, _log_channel, _log_message) \
	Logger::log(_log_channel, _log_level, _log_message, __FILE__, __LINE__);
// a bunch of other macros omitted ...
```

We need some macros to configure Annileen applications as well. I don't know how to do this without using headers (if you do, please let me know :p). Our `definitions.h` header then looks like this:

```c++
#pragma once

#include <memory>

#define ANNILEEN_APP_MAIN(__ApplicationClassName, __ApplicationName) \
    int main(int argc, char* argv[]) { \
        std::unique_ptr<__ApplicationClassName> __app__ = std::make_unique<__ApplicationClassName>(); \
        return __app__->run(__ApplicationName); \
    }

#ifdef _ANNILEEN_COMPILER_EDITOR
    #ifdef ANNILEEN_APPLICATION
        import applicationeditor;
    #endif
    #define ANNILEEN_APP_CLASS_DECLARATION(__ApplicationClassName) class __ApplicationClassName : public annileen::ApplicationEditor
#else
    #ifdef ANNILEEN_APPLICATION
        import application;
    #endif
    #define ANNILEEN_APP_CLASS_DECLARATION(__ApplicationClassName) class __ApplicationClassName : public annileen::Application
#endif // _ANNILEEN_COMPILER_EDITOR
```

### IFCs and external modules [↑](#summary)

In a `.h/.cpp` workflow, if you want to distribute a library you need to provide headers and binaries. In a workflow with modules, you need to provide IFCs and binaries. Every time you compile a module you get a `.obj` and a `.ifc` file. The IFC is a binary file which contains a metadata description of the module interface. In MSVC, if you have everything you need in your solution and the references of the projects are correctly set, then the IFCs will be referenced correctly and everything should work just fine. 

However, if you want to reference external modules, you'll need to specify its IFCs locations (**Configuration Properties > C/C++ > General > Additional BMI Directories**) and names (**Configuration Properties > C/C++ > Additional Module Dependencies**) in the project settings.    
 
### Errors and Intellisense [↑](#summary)

By far the most annoying thing that can happen when you're working with modules is getting the infamous `Internal Compiler Error`. It doesn't tell anything else, just that things went bad, and then you have to play the detective to figure out what is not supported or implemented or what sequence of things triggers the error. 

Another annoying thing in Visual Studio is that Intellisense is not working well yet with modules. Even though things compile and run well, it keeps showing false-positives and those get mixed with real errors. Few examples:
- Let's say you have two classes `A` and `B`, and you define `A` as friend inside `B`. Whenever `A` access private or protected members of `B`, the Intellisense will complain that `Member "something" is inaccessible`.
- You can get `Pointer to incomplete class is not allowed` even though the class is defined and imported. This usually shows up when calling static methods.
- You can get `Type name is not allowed` when using templates. 
Is there a chance of this being me misusing C++ and MSVC? 100%! But the code compiles and runs fine with those errors, sooo ...

### Final considerations [↑](#summary)

The beginning of this migration was very challeging and I thought about giving up a lot of times, however when all the pieces started to work together the experience became very rewarding. Modules can be fun to work with, but there is still a long way before stability. Before joining the C++20 world I couldn't imagine that compilers and build systems were still not so ready for things like modules, given that we are in 2023. [MSVC is feature complete and looks like gcc and clang are almost there](https://en.cppreference.com/w/cpp/compiler_support){:target="_blank"}, but I haven't tried gcc and clang, so I can't tell much about them.

As for build systems, we use premake in Annileen and it's working ok with MSVC. I just needed to set the `cppdialect "C++20"` for projects in the configuration file. Since all files use `.ixx` extension, MSVC automatically recognizes them as modules and does its job. Premake added [some specific options for modules](https://github.com/premake/premake-core/pull/1570){:target="_blank"}, but I haven't tried them yet. 

Things are not so well for generating CMake files (with modules support) with premake yet and this the reason why our implementation of modules is still on a separate branch (we want Annileen compiling on all main systems before merging modules to main branch). [CMake is getting there](https://www.kitware.com/import-cmake-c20-modules/){:target="_blank"} though, but that needs to happen first before premake's turn I guess.

And that's it for now. If you want to check Annileen code and try it at your own risk, [this is the experimental C++20 branch on GitHub](https://github.com/CrociDB/annileen/tree/feature/cpp20){:target="_blank"}.

### References [↑](#summary)

- [Annileen C++20 branch](https://github.com/CrociDB/annileen/tree/feature/cpp20){:target="_blank"}
- [Annileen devlog #0](https://crocidb.com/post/annileen-devlog-0-introduction/){:target="_blank"}
- [Annileen devlog #1](https://crocidb.com/post/annileen-devlog-1-asset-management/){:target="_blank"}
- [MSVC modules documentation](https://learn.microsoft.com/en-us/cpp/cpp/modules-cpp?view=msvc-170){:target="_blank"}
- [Modules documentation on CppReference ](https://en.cppreference.com/w/cpp/language/modules){:target="_blank"}
- [Understanding C++ Modules: Part 1](https://vector-of-bool.github.io/2019/03/10/modules-1.html){:target="_blank"}
- [Understanding C++ Modules: Part 2](https://vector-of-bool.github.io/2019/03/31/modules-2.html){:target="_blank"}
- [Understanding C++ Modules: Part 3](https://vector-of-bool.github.io/2019/10/07/modules-3.html){:target="_blank"}
- [C++ compiler support](https://en.cppreference.com/w/cpp/compiler_support){:target="_blank"}
- [Premake module support PR](https://github.com/premake/premake-core/pull/1570){:target="_blank"}
- [CMake module support as of 2023 Jan.](https://www.kitware.com/import-cmake-c20-modules/){:target="_blank"}
