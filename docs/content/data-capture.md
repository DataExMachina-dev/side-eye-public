---
title: "Data capture"
weight: 30
---

# Data capture

Side-eye data capture operates by navigating the memory, starting from frames -
i.e. currently executed functions, following nested structures and pointers.

To limit the amount of captured data, only configured functions and expressions
are included. Expressions are simply paths to reach specific variable, following
nesting and pointers. Expressions can reference primitive types or structures
(captured as a whole), or more complex types with custom handling.

## Complex types

### Interfaces and any values

When expression references an interface, side-eye will unbox it to discover and
report underlying object type. Additionally, types themselves can be configured
to capture selected expressions, or to capture them as a whole. If the object
behind the interface matches some configured type, it will be captured according
to that type configuration.

### Strings, slices and maps

When expression references a string, a slice, or a map, stored values are captured
with a certain, limited number of elements.

Strings are captured up to first 512 bytes.

For slices, a prefix of elements occupying up to 512 bytes of memory is captured
(or at least one element, if single element is bigger than 512 bytes). Note that
only element structure size counts against the limit, not other objects referenced
by pointers.

For maps, keys-value pairs are captured, from 2048 bytes of hash buckets data
(or at least one bucket - with current go map implementation single bucket contains
8 key-value pairs). Note that unoccupied bucket slots are counted against
the byte limit, so the actual number of captured key-value pairs will depends on
density of the buckets. Selection of buckets is implementation-defined.

### Context [`context.Context`](https://pkg.go.dev/context)

Context capture aims at extracting interesting data from values stored using
[`context.WithValue`](https://pkg.go.dev/context#WithValue) function.

Context capture is configured with list of specifications, each aiming at
extracting one value - specifically the top-most value stored in the context object
that matches that specification.

Each of the specifications may take one of three shapes:

- key and value type is specified - value is captured if the key type matches;
  value type is used to configure interesting expressions
- only value type is specified - value is captured if the value type matches
- only key type is specified - value is captured if the key type matches;
  the value object is captured according to general type capture configuration
  (as with regular interfaces)

Note that only types of keys and values are compared for matching, not the data
they store.
