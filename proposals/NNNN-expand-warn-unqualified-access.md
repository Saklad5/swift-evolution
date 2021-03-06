# Make @warn_unqualified_access applicable to all accessible declarations

* Proposal: [SE-NNNN](NNNN-expand-warn-unqualified-access.md)
* Authors: [Jeremy Saklad](https://github.com/saklad5)
* Review Manager: TBD
* Status: **Awaiting implementation**
* Pitch: [Forum Discussion](https://forums.swift.org/t/make-warn-unqualified-access-applicable-to-all-accessible-declarations/36428)

## Introduction

This proposal seeks to make `@warn_unqualified_access` applicable to most
declarations, allowing for simpler names and improved clarity at the point of
use.

## Motivation

`@warn_unqualified_access` is a useful tool for discouraging ambiguity between
functions with the same name. It does this by requiring the preceding qualifier
(that is, the parent namespace) of the declaration to be explicitly referenced,
and triggering a warning when it is not.

For reasons that are unclear (due to the attribute predating Swift Evolution
entirely), `@warn_unqualified_access` cannot be applied to non-function
declarations. That’s a shame, as they can suffer from a similar issue. The
danger isn’t quite as pronounced without the potential for overloading, but
it is still important. Many programmers emulate the behavior by declaring a
global caseless enumeration with the same name as the module, but I don’t see
why that should remain necessary.

## Proposed solution

`@warn_unqualified_access` should be applicable to almost any declaration. The
current behavior should be maintained: if such a declaration is referenced
without the preceding qualifier, a warning should be triggered.

For example, if a module named `ExampleKit` includes a top-level class called
`Store`, it could be declared with `@warn_unqualified_access` to make it clear
what this class is part of and meant for.

```
@warn_unqualified_access class Store { ... }

let accepted = ExampleKit.Store()

let warned = Store() // Use of 'Store' treated as a reference to class in module 'ExampleKit'
```

## Detailed design

`@warn_unqualified_access` should be applicable to the following declarations.
  * `import`
    * Everything added to the top-level namespace through an import declaration
    should implicitly have `@warn_unqualified_access` applied. This could be
	superseded by more specific import declarations.
  * `let`
  * `var`
  * `typealias`
    * Like access control modifiers, this should apply separately from the type
	being referred to. Unlike access control modifiers, it should not inherit
	`@warn_unqualified_access` from the type it is referring to.
  * `func`
  * `enum`
    * Enumeration cases should not inherit `@warn_unqualified_access`.
    * `case`
	  * This should trigger a warning if the name of the enumeration is not
	  explicitly referenced along with the case.
  * `struct`
  * `class`
    * This should not be inherited by subclasses.
  * `protocol`
    * This should not be inherited by inheriting protocols.

## Source compatibility

Existing source code would not be affected by this change.

## Effect on ABI stability

This change has no effect on the ABI.

## Effect on API resilience

Adding or removing `@warn_unqualified_access` does not and would not affect an
ABI.

## Alternatives considered

There are two common alternatives to this proposal: caseless enumerations and
name prefixes.

### Caseless enumerations

Caseless enumerations are widely used to provide a namespace for declarations
that would otherwise be confusing or conflicting. This proposal does not seek
to supplant all such usage, but it does attempt to improve on one common design
pattern: placing almost every declaration within a top-level caseless
enumeration with the same name as the declaring module.

For example, in a module named `ExampleKit`:

```
enum ExampleKit {
  class Store { ... }
}
```

Such enumerations add no additional information to the type identifier, and
can cause compatibility issues if the module is later renamed. They also make
it difficult to distinguish between module identifiers and the enumeration,
which can become a source of frustration if an enumeration in an unrelated
module shares its name.

### Name prefixes

Name prefixes are ubiquitous in languages such as Objective-C, and remain
fairly common in Swift.

For example, in a module named `ExampleKit`:

```
class ExampleKitStore { ... }
```

While they serve to disambiguate effectively, such names can cause considerable
readability and writability issues. Changing module names can also lead to
massive compatibility issues, and automated systems such as IDEs cannot provide
as much useful context.
