---
description: "Learn more about C++ header units by converting a header file to a header unit"
title: "Walkthrough: Build and import header units in Microsoft Visual C++ projects"
ms.date: "4/13/2021"
ms.custom: "conceptual"
author: "tylermsft"
ms.author: "twhitney"
helpviewer_keywords: ["import", "header unit", "ifc"]
---

# Walkthrough: Build and import header units in Microsoft Visual C++

This article is about building and importing header units using Visual Studio 2019. See [Walkthrough: Import STL libraries using header units](walkthrough-import-stl-header-units.md) to learn specifically how to import Standard Template Library headers as header units.

Header units are the recommended alternative to [precompiled header files](creating-precompiled-header-files.md) (PCH). They're easier to set up and easier to use than a [shared PCH](https://devblogs.microsoft.com/cppblog/shared-pch-usage-sample-in-visual-studio), while providing similar performance benefits. Unlike a PCH, when a header unit changes, only it and its dependencies are rebuilt.

## Prerequisites

Support for header units requires at least Visual Studio 2019 16.10.0 Preview 2.

## What is a header unit

Header units are a binary representation of a header file, and end with an *`.ifc`* extension. This is also the format used for named modules.

An important difference between a header unit and a header file is that header units aren't affected by macro definitions. You can't `#define` a symbol that causes the header unit to behave differently when you import it, the way that you can with a header file.

A similarity is that everything visible from a header file is also visible from a header unit.

Before you can import a header unit, a header file must be compiled into a header unit. An advantage of header units over a PCH is that it can be used in distributed builds. For example, as long as you're using the same compiler to compile the *`.ifc`*  and the program that imports it, and are targeting the same platform and architecture, a header unit produced on one machine can be used on another.

Another advantage of header units over PCHs is that there's more flexibility when it comes to the compiler flags used to compile the header unit and the program that imports it. With a PCH, more compiler flags must be the same. But with header units, the primary flags that should be the same include:

- Exception handling switches such as `/EHsc`
- `/MD[d]` or `MT[d]`
- `/D` You can define additional macros when building the program that imports the header unit, but those used to build the header unit should also be present and defined the same way when building the program that imports the header unit.

## Ways to compile a header unit

There are several ways to compile a file into a header unit:

- **Automatically scan for header units**: This approach is best suited to smaller projects that include many different header files. See [Walkthrough: Import STL libraries as header units](walkthrough-import-stl-header-units.md#approach1) for a demonstration of this approach. The reason it's better suited to smaller projects is because it can't guarantee optimal build throughput since it scans all of the files to find what should be built into header units.

- **Build a shared header unit project**: This approach is best suited for larger projects, and for when you want more control over the organization of the imported header units. You create a static library project (or projects) that contain the header units that you want. Then reference the library project (or projects) from the project that then imports the header units it needs. See [Walkthrough: Import STL libraries as header units](walkthrough-import-stl-header-units.md#approach2) for a demonstration of this approach.

- **Choose individual header units to build**: This approach gives you file by file control over which header files are treated as header units. It's also a good way to quickly and selectively try out header units in your project. This approach is demonstrated in this walkthrough.

## Convert a project to use header units

In this example, you'll compile a header file as a header unit. Begin by creating the following project in Visual Studio:

1. Create a new C++ console app project.
1. Replace the source file contents as follows:
    ```cpp
    #include "Pythagorean.h"
    
    int main()
    {
        PrintPythogoreanTriple(2,3);
        return 0;
    }
    ```
1. Add a header file called `Pythagorean.h`, and replace its contents with the following:
    ```cpp
    #pragma once
    #include <iostream>
    
    void PrintPythogoreanTriple(int a, int b)
    {
        std::cout << "Pythagorean triple a:" << a << " b:" << b << " c:" << a*a + b*b << std::endl;
    }
    ```

To enable header units, first set the **C++ Language Standard** to [`/std:c++latest`](./reference/std-specify-language-standard-version.md):

1. From the Visual Studio main menu, choose **Project** > **Properties**.
1. In the left-hand pane of the project property pages window, select **Configuration Properties** > **General**.
1. Change the **C++ Language Standard** dropdown to **Preview-Features from the Latest C++ Working Draft (/std:c++latest)**.
:::image type="content" source="media/set-cpp-language-latest.png" alt-text="Screenshot showing setting the language standard to preview version.":::

### Compile a header file as a header unit

In the **Solution Explorer**, select the file you want to compile as a header unit. Right-click that file, and select **Properties**. Then do one of the following, depending on the file type:

**For header files**:

Set the **Item Type** property to **C/C++ compiler**. By default, header files have an **Item Type** of **C/C++ header**. Setting this property also sets **C/C++** > **Advanced** > **Compile As** to **Compile as C++ Header Unit (/exportHeader)** for you.
:::image type="content" source="media/change-item-type.png" alt-text="Screenshot showing changing the item type to c/c++ compiler.":::

**For source files** (or header files that don't have a `.h` or `.hpp` extension):

Set the **Compile As** property to **Compile as C++ Header Unit (/exportHeader)**.
:::image type="content" source="media/change-compile-as.png" alt-text="Screenshot showing changing Compile As to Compile as C++ Header Unit.":::

### Change your code to import a header unit

In the source file for the example project, that is, the file that contains `main()`, change `#include "Pythagorean.h"` to `import "Pythagorean.h";` (don't forget the trailing semicolon that is required for `import` statements). When you're compiling a header unit from a system header, use angle brackets (`import <file>;`). If it's a project header, use `import "file";`

Build the solution (**Build** > **Build Solution** from the main menu) and run it to see that it produces the expected output: `Pythagorean triple a:2 b:3 c:13`

In your own projects, repeat this process to compile the header files you want to import as header units.

If you only want to convert a few header files to header units, this is a good approach. But if you have many header files that you want to compile, and the convenience of having the build system automatically handle it outweighs the potential impact on build performance, you can have the build system scan for, and build, header units for you. See [Walkthrough: Import STL libraries as header units](walkthrough-import-stl-header-units.md#approach1) to learn how.

## See also

[Walkthrough: Import STL libraries as header units](walkthrough-import-stl-header-units.md#approach1)\
[Overview of modules in C++](../cpp/modules-cpp.md) \
[`/translateInclude`](./reference/translateinclude.md) \
[`/exportHeader`](./reference/module-exportheader.md) \
[`/headerUnit`](./reference/headerunit.md)