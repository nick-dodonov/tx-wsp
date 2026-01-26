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
- **Custom Rules** - Defined in tx-kit-ext/rules/build/

### Key Build Rules

```starlark
tx_library()   # C++ libraries
tx_binary()    # C++ executables
tx_test()      # C++ tests
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
src/pkg-name/
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
- **uni/tools/** - General tools (build_info, scripts, etc.)

The `tx-kit-ext/` module contains **only Bazel-specific rules and tools**.

## Code Style

- **C++ Standard**: C++20 or later
- **Naming**: PascalCase for types and functions, camelCase for variables
- **Namespaces**: Match directory structure without additional prefixes (e.g., `Boot`, `Http`)
- **Headers**: Use `#pragma once`
- **Documentation**: Doxygen-style comments for public APIs

## Common Tasks

### Building Targets

```bash
# Build specific target
bazel build //src/pkg-name:target

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
- Use `tx_test()` rule in BUILD.bazel
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

**General Tools** (in `uni/` submodule of each module):
- **tools/build_info/** - Build metadata generation (workspace_status.sh)
- **tools/dev/** - Development utilities and scripts

**Bazel-specific Tools** (in `tx-kit-ext/`):
- **rules/build/** - Custom Bazel rules (tx_library, tx_binary, tx_test)
- **tools/dev/init.py** - Development environment setup
- **tools/wasm/** - WebAssembly build support

## Registry Management

Custom modules in `tx-kit-registry/modules/`:
- imgui, SDL3, asio, boost.asio, lwlog, etc.
- Use `tools/update_integrity.py` to update checksums

## Common Pitfalls

- ❌ Don't create standalone BUILD files without proper dependencies
- ❌ Don't use absolute paths in BUILD.bazel
- ❌ Don't forget to run compile_commands refresh after BUILD changes
- ❌ Don't mix language in code (English-only rule)
- ✅ Always specify visibility in cc_library rules
- ✅ Use strip_include_prefix for clean includes
- ✅ Keep BUILD files close to source code

## Getting Help

- Check package README.md files
- Review existing demos in `demo/` directories
- Examine similar packages for patterns
- Consult Bazel documentation for build rules
