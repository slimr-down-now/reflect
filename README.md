# slimr::reflect

Zero-macro C++26 compile-time struct reflection for JSON Schema generation and strict JSON deserialization.

## Why `slimr::reflect`?

In TypeScript, Zod lets you declare a schema once with `.describe("...")` and derive both the static type and the MCP JSON Schema. `slimr::reflect` brings that exact ergonomics to C++26 using `consteval` structured bindings and compile-time string literals — no macros (`NLOHMANN_DEFINE_TYPE_NON_INTRUSIVE`) or external code generators required.

## Install

Add `slimr-down-now/reflect` to `vcpkg-configuration.json`:

```json
{
  "kind": "git",
  "repository": "https://github.com/slimr-down-now/reflect",
  "baseline": "<commit-sha>",
  "packages": [
    "slimr-reflect"
  ]
}
```

Declare `slimr-reflect` in `vcpkg.json` and link `slimr::reflect` in `CMakeLists.txt`:

```cmake
find_package(slimr-reflect CONFIG REQUIRED)
target_link_libraries(my_target PRIVATE slimr::reflect)
```

## Quick Start

```cpp
#include <reflect.hpp>

struct SearchInput {
    slimr::Field<std::string, "Search keywords"> query;
    slimr::Field<std::optional<std::string>, "Folder filter"> folder;
    slimr::OptField<int, "Max results to return"> limit{25};
};

// 1. Generate strict JSON Schema (type: "object", additionalProperties: false)
slimr::Json schema = slimr::schema<SearchInput>();
// -> properties: { query: {type:"string", description:...}, folder: ..., limit: ... }
// -> required: ["query"]

// 2. Deserialize JSON into SearchInput
SearchInput in{};
auto ec = slimr::read_json(in, R"({"query":"kodiaq","limit":10})");

std::string q = in.query.get();                // "kodiaq"
std::string f = in.folder.value_or(" root ");  // " root "
int lim = in.limit.get();                      // 10
```

---

## Field Wrappers

| Wrapper | JSON Schema `required`? | Accessors | Use When |
| --- | --- | --- | --- |
| `slimr::Field<T, "desc">` | Yes (unless `T` is `std::optional<U>`) | `.get()`, `.value`, implicit `const T&`, plus `.has_value()`, `.value_or(def)`, `*f`, `f->` when `T` is `std::optional` | Required tool parameters (or explicitly optional `std::optional<U>` fields) |
| `slimr::OptField<T, "desc">` | No | `.get()`, `.value`, implicit `const T&` | Optional parameters with an inline C++ default value (e.g. `OptField<int, "Limit"> limit{25};`) |
| `slimr::Desc<T, "desc">` | Alias for `Field<T, "desc">` | Same as `Field` | Short alias for `Field` |

## Develop against an unreleased lib

Add `"overlay-ports"` to the consumer's `vcpkg-configuration.json`, then run `just test` or `just build` as usual (add one entry per unreleased lib):

```json
{
  "overlay-ports": [
    "<relative path to ~/dev/slimr/<name>/ports/slimr-<name>>"
  ]
}
```

Remove `"overlay-ports"` and bump the registry baseline once the lib is released. Never commit or release with `overlay-ports` set.

## Environment Variables

| Name | Required | Purpose |
| --- | --- | --- |
| *(none)* | — | — |

## Development

See [docs/development.md](docs/development.md) for building, testing, linting, and releasing.
