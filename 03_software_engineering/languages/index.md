---
layer: 03_software_engineering
type: index
status: evergreen
created: 2026-03-02
tags: [languages]
---
# Languages

Programming language reference notes — syntax, idioms, and key patterns.
## Go

Compiled, statically typed, concurrency-first language. Well-suited for backend services, CLIs, and data pipelines.

- [[go_basics]] — variables, types, fmt formatting
- [[go_functions]] — signatures, multiple returns, closures, defer, variadic
- [[go_control_flow]] — if/switch/for, range
- [[go_structs]] — struct definition, methods, embedding, memory layout
- [[go_interfaces]] — implicit implementation, type assertions, type switches
- [[go_pointers]] — pointer syntax, pass-by-reference, nil, pointer receivers
- [[go_collections]] — slices, maps, arrays
- [[go_errors]] — error interface, custom errors, iota
- [[go_concurrency]] — goroutines, channels, mutexes
- [[go_packages]] — package system, modules, go toolchain
- [[go_generics]] — type parameters, constraints
## JavaScript

Single-threaded, event-driven, runs in browser and Node.js. Dynamically typed.

- [[js_core_language]] — types, variables, coercions, null/undefined
- [[js_functions]] — arrow functions, closures, scope, `this`
- [[js_objects_and_classes]] — objects, prototypes, classes, inheritance
- [[js_collections]] — arrays, maps, sets, loops
- [[js_async]] — event loop, promises, async/await
- [[js_modules]] — CommonJS, ESM, bundlers
- [[js_runtimes]] — Node.js, npm, Deno, Bun
## TypeScript

JavaScript with static types. Structural type system; compiles to JS.

- [[ts_type_system]] — basic types, inference, any/unknown/never, type aliases
- [[ts_unions_and_intersections]] — union types, intersections, literal types
- [[ts_objects_and_interfaces]] — object types, interfaces, discriminated unions
- [[ts_classes]] — access modifiers, abstract classes, implements
- [[ts_generics]] — generic functions, constraints, type inference
- [[ts_type_narrowing]] — narrowing, type predicates, assertions
- [[ts_advanced_types]] — conditional types, mapped types, infer
- [[ts_utility_types]] — Partial, Required, Pick, Omit, Record, Readonly
- [[ts_enums_and_tuples]] — enums, const enums, tuples
- [[ts_local_dev]] — tsconfig, declaration files, Vite setup
