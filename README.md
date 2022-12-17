# makebindingsforbun

*makebindingsforbun* is a binding generator for [Bun](https://bun.sh/).

Given a small JSON config file that includes a path to a C header file and a path to dynamic library file, it generates a javascript binding library to ease working with the native library.

This project is *experimental*, it has managed to create working bindings for [GLFW](https://www.glfw.org/) and [Vulkan](https://www.vulkan.org/), but there are key features not implemented, and it's highly likely you will find bugs.

Additionally, some key features are missing, like generating typescript type definitions.

## Features

*makebindingsforbun* bindings:
- Export functions with automatic serialization of pointers to structs
- Export `to_C`/`from_C` functions to serialize/deserialize pointer to structs
- Export enum constants 
- Export macro constants

Other features include:
- Support for nested structures, including with pointer indirection
- Symbol/function lazy loading. This allows to work with a binding generated for all the functions defined in the header file, while working with dynamic libraries that don't include all the expected symbols. This is Vulkan's case.
- Safe memory allocation with "Wrapped" pointers

## Usage

To install dependencies:

```bash
bun install makebindingsforbun
pip install pycparser
```

To run:

```bash
bun index.ts my_library_binding_config.json
```

## How it works

The provided header file is parsed with [pycparser](https://github.com/eliben/pycparser) generating an AST in JSON format.

The AST is then processed to detect type definitions and function prototypes.

The last steps consists on generating code to ease working with the detected types and functions.

## Wrapped pointers

Bun GC don't keep track of pointer references, and potentially frees memory still referenced via a JS "pointer" or in use by the native library.

To overcome this, the `to_C` functions return a "wrapped" pointer, this is an object consisting of a `ptr` property and a `free` method. This blocks the GC to free the memory until the `free` method is called. Not calling `free` leaks the memory.

Internally, this is achieved by adding a reference to the `ArrayBuffer` in the global context, and removing the reference when `free` is called.


