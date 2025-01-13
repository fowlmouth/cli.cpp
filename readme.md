# cli.cpp - A Minimalistic C++ Command-Line Parsing Library

`cli.cpp` is a small, single-source-file C++ library designed to parse command-line arguments in a straightforward, modern manner. It provides a concise interface to register expected flags and options along with their callbacks, and then seamlessly parse the command line.

This library allows you to:

1. **Register callbacks** for specific flags or options (with or without values).
2. **Capture “free” arguments**—those that don’t correspond to a particular flag.
3. **Bind boolean variables** to flags to detect their presence.

---

## Table of Contents
- [Features](#features)
- [Installation](#installation)
- [API Reference](#api-reference)
- [Example Usage](#example-usage)
- [License](#license)

---

## Features

1. **Modern C++**: Uses `std::function` and `std::string_view` to provide clean, readable code.
2. **Single-source**: All the code is contained in a single `.cpp` (and corresponding `.h`) file for easy integration.
3. **Lightweight**: Minimal overhead and a straightforward interface.
4. **Flexible**: Offers multiple ways to respond to flags—via function callbacks, bound booleans, or capturing free arguments.

---

## Installation

1. Copy the `cli.cpp` and `cli.h` files into your project.
2. Include `cli.h` in your source code
3.  Compile and link `cli.cpp` along with the rest of your project.

Quick Start

1. Create a `CLI` instance in your `main` (or wherever you parse arguments).
2. Register your expected flags/options and what to do when they’re encountered.
3. Call parse with argc and argv.
4. Act on parsed results.

A minimal example (showing just the structure):

```c++
#include <iostream>
#include "cli.h"

int main(int argc, const char** argv)
{
    CLI cli;

    // Register a flag callback
    cli.on("--verbose", []() {
        std::cout << "Verbose mode enabled." << std::endl;
    });

    // Parse command line
    cli.parse(argc, argv);

    return 0;
}
```

## API Reference

The core of the library is the CLI class, which provides methods to register callbacks or bind variables to command-line options. Here’s a quick reference of the main methods:

```c++
class CLI
{
public:
  /*
   * Registers a command-line option that requires a value.
   * E.g. `--file filename`
   */
  CLI& on(const char* arg, std::function< void(std::string_view) > callback);

  /*
   * Registers a command-line option that does NOT require a value.
   * E.g. `--help`
   */
  CLI& on(const char* arg, std::function< void() > callback);

  /*
   * Binds a command-line option (which does NOT require a value)
   * to a boolean reference. The boolean is set to `true` if the flag
   * is encountered.
   */
  CLI& on(const char* arg, bool& flag_present);

  /*
   * Registers a callback for “free” arguments (i.e. those not
   * matching any registered option).
   */
  CLI& on_argument(std::function< void(std::string_view) > callback);

  /*
   * Parses the command-line arguments using the rules/handlers
   * defined above.
   */
  void parse(int argc, const char** argv) const;
};
```

1. `CLI& on(const char* arg, std::function< void(std::string_view) > callback)`
    * Use Case: When the flag/option requires a value (e.g., --file myfile.txt).
    * Example:
        ```c++
        cli.on("--file", [&](std::string_view filename) {
            std::cout << "File: " << filename << std::endl;
        });
        ```
2. `CLI& on(const char* arg, std::function< void() > callback)`
    * Use Case: When the flag does NOT require a value (e.g., --help).
    * Example:
        ```c++
        cli.on("--help", []() {
            std::cout << "Showing help..." << std::endl;
        });
        ```
3. `CLI& on(const char* arg, bool& flag_present)`
    * Use Case: Convenient way to set a boolean to true if a flag is present (e.g., --verbose).
    * Example:
        ```c++
        bool verbose = false;
        cli.on("--verbose", verbose);

        // Later in the code:
        if(verbose)
        {
            std::cout << "Verbose flag was set!\n";
        }
        ```
4. `CLI& on_argument(std::function< void(std::string_view) > callback)`
    * Use Case: Capture extra positional or “free” arguments that don’t match any of the registered flags/options.
    * Example:
        ```c++
        cli.on_argument([](std::string_view arg) {
            std::cout << "Free argument: " << arg << std::endl;
        });
        ```
5. `void parse(int argc, const char** argv) const`
    * Use Case: Parses the command-line arguments, matching them to callbacks or the default argument handler.
    * Behavior: Splits arguments. Triggers the appropriate callback for each recognized option. Calls the argument_callback for any argument not matched to a flag.

## Example Usage

Below is a more comprehensive example showing how you might use all the features together:

```c++
#include <iostream>
#include <string>
#include "cli.h"

int main(int argc, const char** argv)
{
    // Some variables to store parsed info
    bool show_help = false;
    bool verbose = false;
    std::string file_name;

    // Create the CLI object
    CLI cli;

    // Bind a flag to a bool reference
    cli.on("--help", show_help);

    // Bind another flag to a bool reference
    cli.on("--verbose", verbose);

    // Register an option that expects a value
    cli.on("--file", [&](std::string_view value) {
        file_name = std::string(value);
    });

    // Capture any arguments that are NOT recognized as options
    cli.on_argument([&](std::string_view arg) {
        std::cout << "Encountered free argument: " << arg << std::endl;
    });

    // Parse the command line
    cli.parse(argc, argv);

    // After parsing, take action
    if(show_help)
    {
        std::cout << "Usage: my_program [options] [args...]\n"
                  << "  --help     Show this help message\n"
                  << "  --verbose  Enable verbose mode\n"
                  << "  --file     Specify a file name\n";
    }

    if(verbose)
    {
        std::cout << "Verbose mode is ON\n";
    }

    if(!file_name.empty())
    {
        std::cout << "File specified: " << file_name << std::endl;
    }

    // ... rest of your program logic ...
    return 0;
}
```

Compile and run:

```
$ g++ -std=c++17 -o my_program cli.cpp main.cpp
$ ./my_program --verbose --file readme.md extra_arg
Verbose mode is ON
File specified: readme.md
Encountered free argument: extra_arg
```

## License

This library is distributed under the terms of the MIT License. See LICENSE for details.

Happy hacking! If you find any issues or have suggestions, feel free to open an issue or submit a pull request. Contributions are always welcome.
