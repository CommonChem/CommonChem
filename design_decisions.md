# CommonChem Design Decisions

## Introduction

This document aims to provide more detailed explanations behind the various design decisions in CommonChem.

## The name 'CommonChem'

There are already a number of existing 'Chemical JSON' formats. It is inevitable that this term will come to describe a class of formats rather than one specific format. So having 'CommonChem JSON' allows us to easily specify that a file is chemical JSON that adheres to the CommonChem specification. This also allows us to have 'CommonChem YAML', 'CommonChem MessagePack', etc.

It also leaves open the possibility that there could be a CommonChem community effort to produce an open specification for 'CommonChem SDF' or 'CommonChem SMILES'.

It fits quite nicely with the [CommonMark](http://commonmark.org/) markdown specification.

## Supported Serialization Formats

The CommonChem specification aims to be agnostic about serialization formats, however JSON is considered the primary format for a number of reasons:

- It is probably the most widely used format, and is supported natively by many programming languages.

- It provides a minimum required set of features for a serialization format to be suitable for CommonChem (key-values pairs, with string, number, boolean, and array types).

- It provides a maximum required set of features for anything contained within the CommonChem specification. No part of the specification may rely on a feature of a serialization format that is not available in JSON. For example, YAML supports comments, anchors, and custom data types, none of which may be relied upon by CommonChem.

- Recommending that toolkit developers prioritize JSON support, while encouraging users to favor the JSON format, should make it easier to achieve the goal of wide interoperability.

In addition to JSON, toolkits should aim to support YAML and MessagePack also. Both formats are easily derived from a JSON representation. YAML has the benefit of being more easily human-readable, while MessagePack is a more efficient binary format.

## File Extensions

The recommendation is to just use the default file extension of the serialization format. For example, use `.json` for CommonChem JSON. This reflects the fact that CommonChem is just a schema for the contents of the file, not a unique serialization format itself.

If there is some important reason why a more specific file extension is required, the serialization format extension may be prefixed with `cc`. For example, use `.ccjson` for CommonChem JSON.

Another possibility would be to use a single file extension (`.cchem`?) for all CommonChem files, regardless of serialization format. However, this is hard to recommend because it is more likely to lead to confusion about compatibility between files and toolkits.

## Field Names Style Guide

There is no strictly enforced style for field names. Fields in the core CommonChem specification should be lowercase with words separated by underscores as necessary to improve readability. This matches the [PEP8 style guide for Python](https://www.python.org/dev/peps/pep-0008/). CamelCase is discouraged, but may be used in toolkit-specific extensions to match the convention used by the toolkit.

Legibility is prioritized over brevity. CommonChem isn't trying to be the most concise possible representation (there are much better formats for that) so why sacrifice human-readability? For example, we use field names like `bonds` instead of `b`, and `order` instead of `o`, etc.

## Versions

Semantic versioning allows future versions of the CommonChem specification to introduce both backwards-compatible and backwards-incompatible changes in a clear and easily supportable way.

## Multiple Molecules

CommonChem has a top level `molecules` array field to allow multiple molecules to be included in a single file. The most popular chemical file format, MOl/SDF, allows for multiple molecules, therefore CommonChem should also support this. This also allows for greater extensibility than a file with the contents of a single molecules at the root level. In future, new top level fields for other object types such as `reactions` could be added.

## Objects vs. Arrays

There are two potential ways for specifying data that relates to all the atoms in a molecule. One way is to use a separate array for each property, such that the properties for a given atom can be obtained by looking at the same index in each array:

```json
{
  "name": "Acetate",
  "atoms": {
    "elements": [6, 6, 8, 8],
    "hcounts": [3, 0, 0, 0],
    "charges": [0, 0, 0, -1]
  }
}
```

Alternatively, a single array of atom objects that have property fields for each atom:

```json
{
  "name": "Acetate",
  "atoms": [
    {"element": 6, "hcount": 3},
    {"element": 6},
    {"element": 8},
    {"element": 8, "charge": -1},
  ]
}
```

We chose the latter method for a number of reasons:

- It matches better with many applications internal data structures.
- It doesn't require lots `0` or `null` values when many atoms have the default value.
- It allows for sub-object querying in object-oriented databases.
- It doesn't preclude the use of arrays for some properties

On the whole, objects with field names that explicitly define the meaning of values are preferred over arrays of values. For example, coordinates are specified using `x`, `y`, and `z` fields, rather that just an array of three values.

## IDs vs. Indices

Where possible, use the index of objects in an array to refer to them instead of requiring an ID is present on each object. For example, bond `start` and `end` atoms are referred to by the index of the atom in the `atoms` array. For more complicated types of reference, it should be possible to use [JSONPath](http://goessner.net/articles/JsonPath/) notation. 

Many objects in the specification allow for an ID to be specified, but really this is just a convenience to facilitate interaction with an external system or database that requires identifiers.

## Atomic Numbers vs. Element Symbols

Atom elements should be specified using their atomic number. In theory, CommonChem could allow specifying elements by either their atomic number or element symbol, but it is better to pick one way

- Don't need to wait for new symbols to be assigned to new elements
- All toolkits should contain a periodic table that allows conversion from number to symbol and vice versa.

## Index vs. ID For Atom References

Other objects that contain a reference to a specific atom could use either the index of that atom in the `atoms` array, or the value of an `id` field on that atom. We use the index rather than the `id` because it allows the `id` field to be optional.

## Valence Model

Always require explicit hydrogen count? Even on common organic subset? Would help avoid mistakes by implementations that may assume the wrong valence model.

The bonds, charge, radical, and hydrogen count must all be specified for every atom. If any is omitted, the default value is assumed to be zero. It is not deduced by a valence model.

## Allowed Valence States

Some toolkits are more strict, others allow exotic or impossible valence states. CommonChem places no restrictions on allowed valence states. This introduces the possibility that a certain toolkit will not be able to read a certain CommonChem file, but removes the possibility that a user is unable to generate a CommonChem file for a molecule they consider valid. 

## Multi-center Bonds

There is currently poor toolkit support for multi-center bonds, and it complicates many algorithms and data structures. As a result, the Core CommonChem Specification requires that all bonds have a single start atom and a single end atom. Most multi-center bonds can be represented in this way through the use of zero-order bonds.

The Resonance Extension provides a layer on top of the core representation that allows for multi-center bonds.

## Macromolecules

It is important that CommonChem is able to support biological macromolecules, so no aspect of the design should place any unnecessary limit on the number atoms allowed in the molecule.

## Stereochemistry

The most basic tetrahedral stereochemistry indicator would be a `parity` field on atom objects. A clockwise/anti-clockwise value combined with the order of atoms in the molecule's `atoms` array would be enough to give the stereochemistry configuration. However, there's a lot of implicit information when doing it this way. What if there is a `hcount` implicit hydrogen on the stereocenter? What if a lone pair is involved?

This also doesn't help much with more exotic stereochemistry configurations.

An alternative strategy is a separate stereo layer:

```json
{
  "atoms": [...],
  "bonds": [...],
  "stereo": {
    "tetrahedral": [
      {"center": 0, "top": 1, "bottom": 2, "above": 3, "below": 4, "parity": "clockwise"}
    ],
    "planar": [
      {"left": 0, "right": 1, "ltop": 3, "lbottom": 4, "rtop": 5, "rbottom": 6, "parity": "trans"}
    ]
  }
}
```

This is a bit verbose. It allows for categories of stereochemistry type (`tetrahedral`, `planar`), each of which can have different field requirements.

## Extension Field Names

[Reverse domain name notation](https://en.wikipedia.org/wiki/Reverse_domain_name_notation) was considered:
```json
{
  "com.chemaxon.marvin.myproperty": "value"
}
```

But this is a bit verbose, and having periods in the field name is problematic for languages or notations that use them to access nested object fields. Also, not every extension will have a registered domain name to use.

Instead, we use the prefix `x-`.

```json
{
  "x-myextension-myproperty": "value"
}
```

This is similar to the convention for custom HTTP headers, and is also used the [OpenAPI Specification](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.1.md#specificationExtensions).

## The `extensions` Field

Applications that read and write CommonChem files must support an extension in its entirety, or not at all. It is not allowed to interpret a specific field from an extension but not others.

Each extension used by a file should be specified in the top-level `extensions` field.

Each extension should have a publicly available specification that defines its fields. The version number in the `extensions` field corresponds to the version of this specification. For extensions that are linked to a specific toolkit, the extension specification version should be unrelated to the version of the toolkit itself.

## Fields outside the scope of a single molecule

Having top-level fields outside of a single molecule is likely to be beyond the ability of many current toolkits to handle. Molecule-level (and atom-level and bond-level) generic properties have widespread support. But having something like a `name` field at the top level of a file will probably be problematic for many toolkits to roundtrip. So a compromise is to not have any fields outside of molecules in the CommonChem Core Specification, but allow extensions to place fields in the top-level container.
