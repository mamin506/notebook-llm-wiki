---
title: "Protocol Buffer Rules"
category: "reference"
level: "intermediate"
status: "seedling"
sources: ["Protocol Buffer Rules.md"]
tags: ["#protobuf", "#proto", "#code-generation", "#cross-language"]
related: ["[[languages/python]]", "[[languages/cpp]]", "[[reference/general-rules]]"]
last_updated: "2026-07-19"
graph-group: "reference"
---

# Protocol Buffer Rules

Protocol Buffer rules generate language-specific code from `.proto` files, enabling cross-language serialization and RPC.

## Overview

Protocol Buffers (protobuf) is a method of serializing structured data developed by Google. Bazel can generate code for protobuf messages in multiple languages from shared `.proto` files.

**Typical workflow:**
1. Write `.proto` file(s) describing message schemas
2. Create `proto_library` rule
3. Create language-specific rules that depend on the proto library
4. Build code; language-specific code is generated automatically
5. Link generated code into your application

## Core Rules

### proto_library

Declares a collection of protocol buffer definitions.

**Attributes:**
- `srcs`: `.proto` files

**Example:**
```python
proto_library(
    name = "user_proto",
    srcs = ["user.proto"],
)
```

This rule doesn't generate code itself; it's a base for language-specific rules.

### cc_proto_library

Generates C++ code from `.proto` files.

**Attributes:**
- `deps`: Must point to `proto_library` rules

**Example:**
```python
proto_library(
    name = "user_proto",
    srcs = ["user.proto"],
)

cc_proto_library(
    name = "user_cc_proto",
    deps = [":user_proto"],
)

cc_library(
    name = "user_service",
    srcs = ["service.cc"],
    hdrs = ["service.h"],
    deps = [
        ":user_cc_proto",  # Generated C++ code
        "@com_google_protobuf//:protobuf",
    ],
)
```

**Generated Code:**
- `user_proto.pb.h` — Message class definitions
- `user_proto.pb.cc` — Message implementations
- Automatically linked into dependent `cc_library` and `cc_binary` targets

### java_proto_library

Generates Java code from `.proto` files (full protobuf with reflection).

**Attributes:**
- `deps`: Must point to `proto_library` rules

**Example:**
```python
proto_library(
    name = "user_proto",
    srcs = ["user.proto"],
)

java_proto_library(
    name = "user_java_proto",
    deps = [":user_proto"],
)

java_library(
    name = "user_service",
    srcs = ["UserService.java"],
    deps = [
        ":user_java_proto",
        "@com_google_protobuf//:protobuf_java",
    ],
)
```

### java_lite_proto_library

Generates lightweight Java code from `.proto` files (no reflection, smaller runtime).

**Use when:**
- Building for Android (smaller binary)
- Reflection not needed
- Faster runtime performance preferred

**Example:**
```python
proto_library(
    name = "message_proto",
    srcs = ["message.proto"],
)

java_lite_proto_library(
    name = "message_lite_java_proto",
    deps = [":message_proto"],
)

# Android app dependency
android_library(
    name = "app",
    srcs = ["App.java"],
    deps = [
        ":message_lite_java_proto",
    ],
)
```

### py_proto_library

Generates Python code from `.proto` files.

**Attributes:**
- `deps`: Must point to `proto_library` rules

**Example:**
```python
proto_library(
    name = "config_proto",
    srcs = ["config.proto"],
)

py_proto_library(
    name = "config_py_proto",
    deps = [":config_proto"],
)

py_binary(
    name = "app",
    srcs = ["app.py"],
    deps = [
        ":config_py_proto",
    ],
)
```

## Multiple File Organization

### Shared Proto Definitions

```
protos/
├── BUILD
├── common.proto
├── user.proto (imports common.proto)
└── order.proto (imports common.proto)

# BUILD:
proto_library(
    name = "common_proto",
    srcs = ["common.proto"],
)

proto_library(
    name = "user_proto",
    srcs = ["user.proto"],
    deps = [":common_proto"],  # Depends on other proto_library
)

cc_proto_library(
    name = "user_cc_proto",
    deps = [":user_proto"],  # Generates C++ for user.proto and common.proto
)
```

### Cross-Package Proto Dependencies

```
# foo/proto/BUILD
proto_library(
    name = "foo_proto",
    srcs = ["foo.proto"],
)

# bar/proto/BUILD
proto_library(
    name = "bar_proto",
    srcs = ["bar.proto"],
    deps = ["//foo/proto:foo_proto"],  # Cross-package dependency
)

cc_proto_library(
    name = "bar_cc_proto",
    deps = [":bar_proto"],
)
```

## Proto File Syntax

### Proto3 vs Proto2

Bazel supports both proto versions. Proto3 is recommended for new projects.

**Proto3 example:**
```protobuf
syntax = "proto3";

package myapp;

message User {
    string id = 1;
    string name = 2;
    int32 age = 3;
}

service UserService {
    rpc GetUser(UserId) returns (User);
}
```

## Common Patterns

### Separating Proto Definitions from Implementation

```python
# foo/api/BUILD
proto_library(
    name = "foo_proto",
    srcs = ["foo.proto"],
    visibility = ["//visibility:public"],
)

# foo/impl/BUILD
cc_library(
    name = "foo_impl",
    srcs = ["foo_impl.cc"],
    hdrs = ["foo_impl.h"],
    deps = [
        "//foo/api:foo_proto",
        "@com_google_protobuf//:protobuf",
    ],
)
```

### Generated Code Integration

Generated code is automatically linked; you just depend on the language-specific proto rule:

```python
cc_binary(
    name = "app",
    srcs = ["main.cc"],
    deps = [
        ":user_cc_proto",  # Automatically includes generated .pb.h and .pb.cc
        ":user_service",
    ],
)
```

### Cross-Language Serialization

One set of proto definitions, multiple language implementations:

```python
proto_library(
    name = "message_proto",
    srcs = ["message.proto"],
)

cc_proto_library(name = "message_cc", deps = [":message_proto"])
java_proto_library(name = "message_java", deps = [":message_proto"])
py_proto_library(name = "message_py", deps = [":message_proto"])

# Services can be in different languages but share message format
cc_binary(
    name = "service_cpp",
    srcs = ["service.cc"],
    deps = [":message_cc"],
)

py_binary(
    name = "service_py",
    srcs = ["service.py"],
    deps = [":message_py"],
)
```

## Troubleshooting

**Issue: "Proto file not found" or "Cannot import proto"**
- Check proto file paths are correct
- Use proper Bazel labels in `srcs`
- For imports in `.proto`, use relative or absolute package paths

**Issue: "Symbol not found" in generated code**
- Ensure language-specific proto rule (`cc_proto_library`, `java_proto_library`, etc.) is in `deps`, not just the base `proto_library`
- Link against protobuf runtime: `@com_google_protobuf//:protobuf`, `@com_google_protobuf//:protobuf_java`

**Issue: "Generated files not available"**
- Don't try to reference `.pb.h` or `.pb.cc` files directly; they're generated
- Depend on the proto rule; generated code is automatically linked
- Generated code is in Bazel's internal build directories, not your source tree

**Issue: Changing `.proto` file doesn't regenerate**
- Cache invalidation happens automatically when `.proto` changes
- Try `bazel clean` if stale code persists

## See Also

- [[languages/python]], [[languages/cpp]], [[languages/java]] — Language guides
- [[reference/general-rules]] — Utility rules
- [[patterns/dependency-management]] — Dependency patterns
