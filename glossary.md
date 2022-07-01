# Glossary

As with most software projects, terms are often overloaded. The goal of this document is to provide
an unambiguous definition of terms frequently used by SIG Registries.

## Terms

### Annotations

Annotations are typed metadata added to a package after its creation. For example, "yank" and "takedown" annotation types.

### Component

A component is defined by the (emerging) [W3C WebAssembly Component Model specification](https://github.com/WebAssembly/component-model) which defines a component as a portable binary built from WebAssembly core modules with statically-analyzable, capability-safe, language-agnostic interfaces.

A component package is a type of [package](#package) whose contents are a component.

### Bundle and Bundling

A bundle or a component that has been "bundled" is the result of when a [component](#component) has only interface dependencies and can thus run directly on a wasm engine that natively implements those interfaces without any registry access. A bundle function replaces a [component](#component)’s [imports](#imports) with inline copies via local module definitions.

This has also been referred to as producing a composite of a component's DAG of dependencies.

### Entity

Entity is the name that replaces [package](#package) in most contexts. An entity is interchangeable with package.

### Exports

Exports are the names of [interfaces](#interfaces) that a component defines, e.g. `wasi:cache`.

### Imports

Imports are names of [interfaces](#interfaces) that a component depends on, e.g. `wasi:cache`.

### Interface

An interface is a pair of a string and an instance type that can be imported or exported. Interfaces are a component's API surface ([exports](#exports)) as well as the API's that a component ([imports](#imports)).

### Module

A common question is "What is the difference between a component and a module?". A Wasm module is a core Wasm module and compatible with Wasm 1.0 whereas a component adheres to the evolving Component Model specification with support for interfaces definitions (beyond i32's in core Wasm).

### Namespace

A namespace is a named-definition of scope. A registry instance defines a disjoint namespace such that no registry instance's package names ever shadow or backstop another.

### Package

A package is type of content bundle uploaded to the registry. The registry architecture defines a number of built-in package types. Some of the package types include [Component](#component), [Interface](#interface), [Profile](#profile), and [Library](#library) packages.

### Policies

The registry architecture provides mechanisms for registry instances to apply their own appropriate policies.

### Profile

A profile is a collection of interfaces that are imported and exported by to the components they execute.

### Provenance

Provenance is a record of ownership of a package. The state of a registry is provenantial when it is *internally consistent* and every package release has provenance.

### Registry

There isn’t just one registry: there is a single registry architecture which consists of a common set of tools and building blocks, and many registry instances, which are live services implemented in terms of the registry architecture. Registry instances can be general and global or specific to individual projects, companies, teams or accounts.

### Signatures

Signatures are cryptographic bindings to signing identities.

### Yank

Yank is an assertion by the [publisher](#publisher) that "this release is not fit for use". When a package is "yanked", the release is not altered but the release may be excluded from default query results. An example of when a package might be "yanked" is after an accidental or unintentional release.

### Takedown

Takedown is an assertion by the [publisher](#publisher) or registry operator that "this release was removed for legal or policy reasons". This is unique from [yank](#yank)'s behavior in that the release's content URLs and potentially release metadata are removed from the registry. The primary use-case for a takedown is for *legal* reasons (DMCA et al).

## Governance and community terminology

### Phases of agreement

Phases of agreement are the levels of agreement required to advance proposals to the next stage. For more details, see [phases.md](phases.md).
