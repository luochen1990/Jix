Jix Programming Language
========================

Jix is a pure functional configuration generation language similar to Nix, which is a superset of JSON and supports lambda functions, static typing, data merging, and partial evaluation.

The first point that distinguishes it from the Nix language is its type system, which offers first-class support for a mergable module system, which can be used as an alternative to the NixOS module system;

The second point is its partial evaluation capability, which allows you to write code at any level of abstraction and fuse it to a lower level of abstraction, thereby simultaneously meeting the requirements for code maintainability (which demands coding at a high level of abstraction) and auditability (which requires coding at a low level of abstraction).
