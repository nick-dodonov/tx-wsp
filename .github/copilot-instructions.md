# TX Workspace Development Guidelines

## Language Requirements

**CRITICAL**: All code artifacts must be in English:
- Source code comments
- Documentation (README, inline docs)
- Log messages and string literals
- Commit messages
- Variable/function/class names

User communication with the agent can be in any language, but **all generated code and documentation must be English-only**.

## Workspace Structure

This is a multi-module Bazel workspace:

- **tx-pkg-aux** - Core packages (app, boot, http, log)
- **tx-pkg-misc** - Miscellaneous packages and demos
- **tx-kit-ext** - Build rules and development tools
- **tx-kit-registry** - Local Bazel module registry
- **tx-app-demo** - Demo applications

## Build System

- **Bazel** - Primary build tool
- **Module System** - Uses Bazel modules (MODULE.bazel)
- **Custom Rules** - Defined in tx-kit-ext/rules/

### Key Build Rules

```starlark
multi_lib()   # C++ libraries
multi_app()   # C++ executables
multi_test()  # C++ tests
```

## Development Workflow

### Adding New Features

1. **Create/modify source files** in appropriate module
2. **Update BUILD.bazel** with dependencies
3. **Build** using Bazel: `bazel build //path/to:target`
4. **Test** if applicable: `bazel test //path/to:test`

### Working with Packages

Each package follows structure:
```
src/pkg/name/
├── BUILD.bazel
├── Namespace/
│   ├── Header.h
│   └── Implementation.cpp
└── README.md
```

### Build Configuration

Bazel configuration files and general-purpose tools are located in the `uni/` submodule within each top-level module:
- **uni/.bazelrc** - Module-specific Bazel configuration
- **uni/user.pre.bazelrc** - User-specific pre-build config
- **uni/user.post.bazelrc** - User-specific post-build config
- **uni/tools/** - General tools that cannot be built with Bazel (build_info generator)

The `tx-kit-ext/` module contains Bazel-specific rules and special tools for development.

## Code Style

- **C++ Standard**: C++20 or later
- **Naming**: PascalCase for types and functions, camelCase for variables
- **Namespaces**: Match directory structure without additional prefixes (e.g., `Boot`, `Http`)
- **Headers**: Use `#pragma once`
- **Documentation**: Doxygen-style comments for public APIs; full parameter documentation optional if arguments are self-explanatory

## Modern C++ Features

Leverage modern C++20/23/26 features for cleaner, safer, and more efficient code:

### Core Language Features
- **Concepts** - Use concepts for template constraints instead of SFINAE
- **Coroutines** - Use `co_await`, `co_return`, `co_yield` for asynchronous operations
- **Ranges** - Prefer ranges and views over raw iterators
- **Designated initializers** - Use for struct initialization when it improves clarity
- **Three-way comparison (spaceship operator)** - Implement `operator<=>` for custom types
- **consteval/constinit** - Use for compile-time evaluation and initialization

### Standard Library Features
- **std::span** - Use instead of pointer + size pairs
- **std::string_view** - Prefer over `const std::string&` to avoid unnecessary copies
- **std::expected** - Use for error handling (already in use)
- **std::optional** - Use for values that may not exist
- **std::format** (C++20) / fmt library - For type-safe formatting
- **std::jthread** - Use instead of `std::thread` for automatic joining
- **std::bit_cast** - Use instead of `reinterpret_cast` for type punning when possible

### Smart Pointers & RAII
- **std::unique_ptr** - Default choice for owned pointers
- **std::shared_ptr** - Only when shared ownership is needed
- **std::make_unique/make_shared** - Prefer over raw `new`

### Best Practices
- **Avoid unnecessary copies** - Use move semantics, `std::string_view`, `std::span`
- **Prefer value semantics** - Return by value, rely on copy elision and move semantics
- **Use structured bindings** - `auto [key, value] = map.insert(...)`
- **Range-based for loops** - Prefer over index-based loops
- **if/switch with initializers** - `if (auto result = func(); result.has_value())`

## Code Quality Principles

### DRY (Don't Repeat Yourself)
- **Eliminate duplication** - Extract repeated logic into helper functions or methods
- **Shared constants** - Define magic numbers and strings as named constants
- **Reusable components** - Create abstractions for common patterns

### SOLID Principles
- **Single Responsibility** - Each class/function should have one clear purpose
- **Open/Closed** - Open for extension, closed for modification (use inheritance/composition)
- **Liskov Substitution** - Derived classes must be substitutable for base classes
- **Interface Segregation** - Prefer small, focused interfaces over large ones
- **Dependency Inversion** - Depend on abstractions, not concrete implementations

### Clean Code Practices
- **Small functions** - Keep functions focused and concise (prefer < 30 lines)
- **Meaningful names** - Use descriptive names that reveal intent
- **Early returns** - Return early from error conditions to reduce nesting
- **Const correctness** - Mark variables and methods `const` when they don't modify state
- **Error handling** - Use `std::expected` for recoverable errors; prefer propagating errors over silent failures
- **Resource management** - Use RAII (smart pointers, custom wrappers) for resource lifetime

### Performance Considerations
- **Avoid premature optimization** - Write clear code first, optimize when needed
- **Move semantics** - Use `std::move` for expensive objects when ownership transfers
- **Pass by value for small types** - Use pass-by-value for small types (< 16 bytes)
- **Pass by const reference** - For larger objects that won't be modified (or use `std::string_view`/`std::span`)
- **Pass by value for coroutines** - Coroutine parameters must not be references
- **Avoid unnecessary allocations** - Use `string_view`, `span`, and stack allocations when possible

### Code Organization
- **Header files** - Declare interfaces; include only necessary headers
- **Implementation files** - Define implementations; use forward declarations when possible
- **Private helpers** - Keep implementation details private or in anonymous namespaces
- **Logical grouping** - Group related functions/methods together

## Common Tasks

### Building Targets

```bash
# Build specific target
bazel build //src/pkg/name:target

# Build demo applications
bazel build //demo/hello-boot
```

### Refresh Compile Commands (for IDE)

```bash
bazel run @hedron_compile_commands//:refresh_all
```

### Working with Dependencies

Dependencies declared in `MODULE.bazel`:
- External modules from BCR or local registry
- Local modules via `local_path_override`

## Testing

- Tests located in `test/` directories
- Use `multi_test()` rule in BUILD.bazel
- Run with: `bazel test //test:target_test`

## Best Practices

1. **Keep modules focused** - Single responsibility per package
2. **Minimize dependencies** - Avoid circular dependencies
3. **Document public APIs** - All headers need clear documentation
4. **Write tests** - Cover critical functionality
5. **Use namespaces** - Prevent symbol collisions
6. **Follow workspace structure** - Maintain consistent organization

## Build Info Integration

The workspace uses Bazel's linkstamp feature for build metadata:
- Git SHA, branch, status
- Build timestamp (RFC 3339 format)
- Build user and host

Access via `Build::Info` interface in pkg-build.

## Tools and Extensions

**General Tools and Configs** (in shared `uni/` submodule of each module)
- Bazel configuration files
- Default user-specific pre/post build configs
- General-purpose scripts and tools (e.g., build_info generator)

**Bazel-specific Tools** (in `tx-kit-ext/`) include:
- Custom Bazel build rules
- Scripts to simplify build execution
- Useful tools to setup and develop

## Registry Management

Custom modules in `tx-kit-registry/modules/`:
- imgui, SDL3, asio, boost.asio, lwlog, etc.
- Use `tools/update_integrity.py` to update checksums

## Common Pitfalls

- ❌ Don't mix language in code (English-only rule)
- ❌ Don't create standalone BUILD files without proper dependencies or using absolute paths
- ❌ **ABSOLUTELY FORBIDDEN: `bazel clean --expunge` or `bazel clean --expunge_async`** - These commands delete the entire Bazel cache and cause EXTREMELY long rebuild times (hours). **DO NOT USE THESE COMMANDS UNDER ANY CIRCUMSTANCES** unless explicitly requested by the user. If cleanup is needed, manually remove specific files from `bazel-out/` or `bazel-bin/` instead
- ⚠️  `bazel clean` (without flags) should also be avoided in most cases - prefer targeted cleanup
- ✅ Use strip_include_prefix for clean includes
- ✅ Keep BUILD files close to source code

## Getting Help

- Check package README.md files
- Review existing demos in `demo/` and `test/` directories
- Examine similar packages for patterns
- Consult Bazel documentation for build rules
