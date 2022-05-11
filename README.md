# Library version of druntime AA

This library is a template-library version of the AA implementation used by the d programming language. It is binary compatbile, so reinterpret-casting a `Hash!(K, V)` to/from a `V[K]` is valid.

If any changes are made to the druntime AA implementation, this library may stop being binary compatible (or will be tied to specific versions). The latest D version tested with this library is 2.099.1

## Differences

1. Special mechanisms are not possible with library code. For instance, the outer hash of nested hashes knowing that it is being used for an assignment operation.
2. For now, only `opIndexAssign` is defined for adding a value to a `Hash`. This means things like `hash[unassignedKey]++` will not work.
3. Not all "methods" of AAs are implemented yet, e.g. `require`. These will be added.
4. You can build a `Hash!(K, V)` at compile time, and at runtime use it as a `V[K]` via the `asAA` UFCS method.

## Compatibility/conversion to language AA

The code is built to use the same exact layout and hash implementation as the builtin language AA. There is a difference in that `Hash` uses compile-time information about the types to compute hashes, and uses the language to properly manage lifetimes. The druntime AA implementation is all based on `TypeInfo`, and therefore must handle all these mechanisms manually.

In particular, the AA builds a fake `TypeInfo` for the `Entry` struct that contains a key and value (a thing that has no formal type). Because it is using compile-time introspection, a `Hash` does not use the `TypeInfo` for this information, and therefore doesn't need it.

However, when converting to an AA, it ensures the `TypeInfo` is properly populated, but instead of using a fake `TypeInfo`, the actual `TypeInfo` for the `Entry` struct is used. This *should* be binary compatible with the AA, because the AA's fake is attempting to construct a fake `TypeInfo` that would mimic what a real `Entry` struct's `TypeInfo` would be.

Use the `asAA` UFCS method to extract an associative array from a given hash.

You can just assign a `Hash` from a compatible AA. By compatible, I mean that the AA's keys convert to the `Hash`'s keys, and the AA's values convert to the `Hash`'s values. If the keys and values are the same type, the `Hash` will use the AA's internal representation directly.

## Tests

Tests are sparse at this time. Just enough to ensure the binary compatibility is working, and that the hash works as expected. I plan to add more.

## Future plans

1. Try and ensure all methods/properties are identical to builtin AAs
2. Improve tests
3. flesh out `@safe` and other attributes, make sure they are inferred correctly.
4. Investigate ways to prove the layout/code of the builtin AA has not changed (even if done via CI).
