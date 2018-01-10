---
title: CommonChem Specification
version: 1.0
author:
    - name: Matthew Swain
    - email: m.swain@me.com
---

# CommonChem Specification

## Introduction

This is the specification for CommonChem, a data format for representing chemical information. The format is designed to be easy for both humans and computers to understand.

One of the main features of CommonChem is its extensibility. This means it can accommodate kinds of data that are not explicitly described in this core specification, thus allowing for improved parsing performance because applications can store derived data in the file instead of having to regenerate it.

The purpose of this open, versioned specification is to facilitate interoperability and to make it as easy as possible for software developers to support CommonChem files in their applications. This specification is sometimes referred to as the "Core CommonChem Specification" to distinguish it from extension specifications.

## Table of Contents

- [Versions](#versions)
- [Format](#format)
- [Schema](#schema)
- [Examples](#examples)

## Versions

This specification is versioned with a number that consists of two components: `major.minor` (for example 1.2). The major version is incremented when a backwards-incompatible change is made, and the minor version is incremented when a backwards-compatible change is made. Applications that are designed for a specific version should continue to work with subsequent minor releases, although they will not necessarily fully support any newly added features. Applications should not attempt to read or write a CommonChem file with a major version that they do not support.

Within a CommonChem file, versions are stored using a single integer where 1000 values are allocated for potential minor versions in each major version. For example, `1000` corresponds to version `1.0`, and `2010` would correspond to version `2.10`.

### Version History

| Version | Integer Version | Date       | Notes                                          |
|---------|-----------------|------------|------------------------------------------------|
| 1.0     | 1000            | 2018-??-?? | First version of the CommonChem Specification. |

## Format

- All field names are case-sensitive.
- All field names must be unique within the containing object.
- The order of fields within an object is not significant.
- All fields are optional, unless it is described below as **Required**.
- If the absence of a field implies a default value, this is described.

There are two kinds of fields:

- **Core fields** have a fixed name that is declared in the [schema](#schema) section of this specification.
- **Extension fields** must occur in an [Extension object](#extension-object) and are declared in a separate extension specification.

### Serialization

It should be possible to use CommonChem with any JSON-compatible serialization format. That is, the format must at least support the features of JSON. It may have additional features, but these should not be required by the Core CommonChem Specification or any CommonChem extension specification.

#### JSON

JSON is the primary serialization format that is recommended for most users. All examples in this specification assume JSON format.

JSON example:

```json
{
  "commonchem": 1000,
  "molecules": [{
    "name": "ethane",
    "atoms": [{"z": 6, "impHs": 3}, {"z": 6, "impHs": 3}],
    "bonds": [{"type": 1, "atoms": [0, 1]}]
  }]
}
```

#### YAML

[YAML](http://yaml.org) is a data serialization format that is designed to be more human-readable than JSON. It offers a superset of JSON features, although none of these additional features are explicitly required by CommonChem.

YAML example:

```yaml
commonchem: 1000
molecules:
  - name: ethane
    atoms:
      - z: 6
        impHs: 3
      - z: 6
        impHs: 3
    bonds:
      - type: 1
        atoms: [0, 1]
```

#### MessagePack

[MessagePack](https://msgpack.org) is a binary format that is designed to be more efficient than JSON. It is not human readable, but offers smaller file sizes and faster transmission over the Internet.

### Data Types

The data types used by CommonChem match those supported by JSON.

| Type      | Description                                |
|-----------|--------------------------------------------|
| `object`  | Container with fields and values           |
| `array`   | Ordered list of values                     |
| `string`  | UTF-8 encoded unicode string               |
| `number`  | Numeric type                               |
| `boolean` | `true` or `false`                          |
| `null`    | Explicit null type (as opposed to missing) |

The schema also makes use of some validation formats, that are built on top of the types.

| Format     | Type      | Description                    |
|------------|-----------|--------------------------------|
| `integer`  | `number`  | Number without a decimal point |
| `datetime` | `string`  | Timestamp string (RFC3339?)    |
| `byte`     | `string`  | base64 encoded binary data     |

There are not separate integer and floating point numeric types in JSON. Field with an `integer` type in this specification should always have values that are numbers with no decimal point or fractional part. Internally, toolkits should parse and store these as `int` types. All other numbers should be stored as floating point types.

There is also no dedicated binary data type in JSON. Instead, binary data should be represented as a base64-encoded string of characters.

## Schema

### Container Object

A file that conforms to the CommonChem Specification contains a single object at its root level. The top level object is known as the Container.

Example of a Container object:

```json
{
   "commonchem": 1000,
   "molecules": [{"atoms": [{"z": 6, "impHs": 4}]}]
}
```

It can have the following fields:

| Name       | Type                                  | Description                                                    |
|------------|---------------------------------------|----------------------------------------------------------------|
| commonchem | `integer`                             | Indicates that the file conforms to a specific version of the CommonChem specification. **Required**. |
| defaults   | [Defaults Object](#defaults-object)   | Defines the default values for missing fields.                 |
| molecules  | [[Molecule Object](#molecule-object)] | Array of Molecule objects, each of which describes a chemical structure. |

It is recommended that the `commonchem` field is the first field in the file, but this is solely to help with human-readability.

### Defaults Object

The Defaults object allows the default field values to be defined for various object types, so they don't have to be explicitly specified on every occurrence of that object type.

Example of a Defaults object:

```json
{
  "atom": {"stereo": "unspecified", "chg": 0, "nRad": 0, "Z": 6, "impHs": 0, "isotope": 0},
  "bond": {"stereoAtoms": [], "stereo": "unspecified", "type": 1}
}
```

The following fields are available:

| Name  | Type                        | Description                            |
|-------|-----------------------------|----------------------------------------|
| atom  | [Atom Object](#atom-object) | Atom object with default field values. |
| bond  | [Bond Object](#bond-object) | Bond object with default field values. |

### Molecule Object

A Molecule object describes the structure and properties of a collection of atoms. The atoms may be connected by bonds, but this is not required: Multiple disconnected fragments may be contained in a single Molecule object.

Example of a Molecule object:

```json
{
    "id": "CID6324",
    "name": "ethane",
    "atoms": [
      {"z": 6, "impHs": 3},
      {"z": 6, "impHs": 3}
    ],
    "bonds": [
      {"type": 1, "atoms": [0, 1]}
    ]
}
```

The following fields are available:

| Name        | Type                                    | Description                 |
|-------------|-----------------------------------------|-----------------------------|
| name        | `string`                                | Name for this Molecule.     |
| atoms       | [[Atom Object](#atom-object)]           | Array of Atom objects.      |
| bonds       | [[Bond Object](#bond-object)]           | Array of Bond objects.      |
| conformers  | [[Conformer Object](#conformer-object)] | Array of Conformer objects. |
| properties  | [[Property Object](#property-object)]   | Array of Property objects.  |
| extensions  | [[Extension Object](#extension-object)] | Array of Extension objects. |

### Atom Object

The following fields are available:

| Name    | Type      | Description                                                                               |
|---------|-----------|-------------------------------------------------------------------------------------------|
| z       | `integer` | Atomic number. **Required**.                                                              |
| chg     | `integer` | Formal charge. **Default**: 0.                                                            |
| impHs   | `integer` | Count of implicit hydrogens bonded to this atom. Does not include any hydrogen atoms that exist explicitly in the array of atom objects. **Default**: 0. |
| isotope | `integer` | Atomic mass number. **Default**: 0, which indicates natural abundance mixture.            |
| nRad    | `integer` | Number of radical electrons. **Default**: 0.                                              |
| stereo  | `string`  | Allowed values are `cw`, `ccw`, `unspecified`, `unknown`, or `other`.                     |

In contrast to many other formats, no assumptions are made about allowed valence states, and therefore the hydrogen count must always be specified (if non-zero).

Hydrogens may be represented as actual atom objects in the molecular graph as an alternative to the `impHs` field. This is necessary if:

- Coordinates are provided for the hydrogen (typically in 3-D)
- The hydrogen has a charge or isotope number
- The hydrogen does not have exactly one bond to a non-hydrogen atom

Tetrahedral stereochemistry is specified using the `stereo` field. The order of the bonded atom neighbors is given by their relative order in the molecule's `atoms` array. If the atom has a non-zero `impHs` the hydrogen is considered to be the first atom neighbor in the order. Looking towards the chiral center from the first atom neighbor, the remaining neighbors occur in a clockwise (`cw`) or counter-clockwise (`ccw`) direction.

Atom coordinates are specified using the [`conformers`](#Conformer) molecule field.

### Bond Object

The following fields are available:

| Name         | Type        | Description                                                             |
|-------------|-------------|--------------------------------------------------------------------------|
| type        | `integer`   | Bond order. **Required**.                                                |
| atoms       | [`integer`] | Indices of atoms in the atoms array involved in this bond. **Required**  |
| stereoAtoms | [`integer`] | Indices of the atoms determining the stereochemistry. **Default**: `[]`. |
| stereo      | `string`    | Allowed values are `unspecified`, `cis`, `trans`, `other`.               |

Allowed bond orders are 0, 1, 2, 3. Zero-order bonds do not have any valence contribution, and should be used for things like coordination bonds and hydrogen bonds. Bond orders 1, 2, and 3 correspond to single, double, and triple bonds respectively.

The bond `atoms` array uses the zero-based index of atoms in the molecule's `atoms` array. It must contain exactly two integers.

### Conformer Object

Multiple different molecular conformations may be specified for a single molecule by including an array of Conformer objects.

The following fields are available:

| Name   | Type         | Description                                    |
|--------|--------------|------------------------------------------------|
| dim    | `integer`    | Dimensionality. **Required**.                  |
| coords | [[`number`]] | Array of 2-D or 3-D coordinates. **Required**. |

The `coords` field is an array of arrays, where each of the inner arrays contains either two or three `number` values. All coordinates in a conformer should have the same dimensionality.

Example of a CommonChem file containing conformer coordinates:

```json
{
  "commonchem": 1000,
  "molecules": [
    {
      "atoms": [{"z": 6}, {"z": 6}, {"z": 8}, {"z": 8}],
      "bonds": [{"atoms": [0, 1], "type": 1}, {"atoms": [1, 2], "type": 1}, {"atoms": [1, 3], "type": 2}],
      "conformers": [
        {"dim": 2, "coords": [[1.7321, 0.0000], [0.8660, 0.5000], [0.8660, 1.5000], [0.0000, 0.0000]]}
      ]
    }
  ]
}
```

### Property Object

Property objects allow generic data to be stored on molecules, similarly to an SDF property field.

Example of a property object:

```json
{"name": "PubChem CID", "value": "2244"}
```

The following fields are available:

| Name   | Type     | Description                   |
|--------|----------|-------------------------------|
| name   | `string` | Property name. **Required**.  |
| value  | any?     | Property value. **Required**. |

Applications should not interpret properties to affect chemical representation. Chemically meaningful data should instead use an [Extension object](#extension-object).

### Extension Object

Extension objects contain aspects of chemical representation that are not provided for by the core CommonChem specification.

Example of a CommonChem file containing an extension:

```json
{
  "commonchem": {"version": 1000},
  "molecules": [
    {
      "atoms": [{"z": 6}, {"z": 6}, {"z": 8}, {"z": 8}],
      "bonds": [{"atoms": [0, 1], "type": 1}, {"atoms": [1, 2], "type": 1}, {"atoms": [1, 3], "type": 2}],
      "extensions": [
        {"name": "myextension", "version": 1000, "myproperty": "value"}
      ]
    }
  ]
}
```

| Name       | Type      | Description                                                                         |
|------------|-----------|-------------------------------------------------------------------------------------|
| name       | `string`  | Extension identifier. **Required**.                                                 |
| version    | `integer` | The version of the extension specification that the file conforms to. **Required**. |

The only required fields on an extension object are `name` and `version`. The version number should be an `integer` that follows the [same convention](#versions) as the core CommonChem specification version.

Extensions should not reference or depend on fields in other extensions.

Applications should emit a warning when reading and writing files that contain extensions that they do not support. When reading a file with unsupported extensions, the extension fields should be read and stored as generic properties on the relevant Molecule. When writing back to a CommonChem output file, every effort should be made to preserve the extension data, unmodified from the original input. However, if any modifications are made to any of the structure-determining data fields all extensions should be either be regenerated (if supported) or removed (if not).

For example, consider the situation where one application stores aromaticity information in an extension. If a second application is then used to process a molecule and break a bond in the aromatic ring, the extension data is no longer valid and the application should either regenerate or remove it when writing to a CommonChem output file.

Applications that read and write CommonChem files must support an extension in its entirety, or not at all. It is not allowed to interpret a specific field from an extension but not others. The only circumstances under which an application may write an unsupported extension is when preserving it unmodified from an input.

Anyone may create an extension specification. Submission for review through the CommonChem GitHub repository is encouraged but not required. An approved review has the benefit of reserving the extension `name`, should the situation arise where multiple extensions desire the same `name`. It also serves to publicize the extension to application developers, encouraging support in applications.

Two main use cases are envisioned for extensions: **Application Extensions** and **Shared Extensions**.

#### Application Extensions

Application extensions should only be used for data where it does not make sense for any other application to interpret it. Applications are discouraged from actually interpreting and making use of application-specific extensions written by other applications. They should be read in and written out without modification, by assigning the values as generic properties to the corresponding Molecule object.

A specification should still be published for application extensions, even if no other applications will implement it. This serves to reserve the extension `name` and provide a public record of the extension's existence.

#### Shared Extensions

One of the goals of the CommonChem project is for as many applications as possible to achieve full support for the Core CommonChem Specification. Therefore, data that is unlikely to be supported by the majority of applications is considered outside the scope of the core specification and should instead be defined in a shared extension that a subset of applications may support.

## Examples

```json
{
  "commonchem": 1000,
  "defaults": {
    "atom": {"stereo": "unspecified", "chg": 0, "nRad": 0, "Z": 6, "impHs": 0, "isotope": 0},
    "bond": {"stereoAtoms": [], "stereo": "unspecified", "type": 1}
  }
  "molecules": [
    {
      "name": "example 3", 
      "atoms": [
        {"z": 8},
        {"z": 6, "stereo": "ccw"},
        {"z": 17},
        {"z": 6},
        {"z": 6},
        {"z": 6},
        {"z": 1},
        {"z": 1},
        {"z": 1},
        {"z": 1},
        {"z": 1},
        {"z": 1},
        {"z": 1}
      ],
      "bonds": [
         {"atoms": [1, 2]},
         {"atoms": [1, 3]},
         {"atoms": [3, 4], "type": 2, "stereoAtoms": [1, 5], "stereo": "trans"},
         {"atoms": [4, 5]},
         {"atoms": [0, 6]},
         {"atoms": [1, 7]},
         {"atoms": [3, 8]},
         {"atoms": [4, 9]},
         {"atoms": [5, 10]},
         {"atoms": [5, 11]},
         {"atoms": [5, 12]}
      ],
      "conformers": [
        {"dim": 2, "coords": [[-2.7796, 0.9135], [-1.6349, -0.0559], [-2.7883, -1.0149], [-0.2231, 0.4508], [0.9216, -0.5186], [2.3335, -0.0119], [-4.1914, 0.4068], [-0.8908, -1.3583], [0.0441, 1.9268], [0.6545, -1.9946], [3.4782, -0.9813], [3.4869, 0.947], [1.5894, 1.2905]]},
        {"dim": 3, "coords": [[2.1105, 0.4157, -0.2114], [1.5717, -0.5232, 0.443], [2.3766, -2.0449, 0.0206], [0.1184, -0.6373, 0.0945], [-0.5623, 0.4623, -0.2583], [-2.0473, 0.4396, -0.2488], [2.572, 1.0572, -0.2412], [1.7018, -0.2628, 1.5244], [-0.3963, -1.593, 0.0418], [-0.1015, 1.3944, 0.0865], [-2.4928, 1.4231, 0.0277], [-2.3664, -0.3192, 0.4839], [-2.4844, 0.1881, -1.2369]]}
      ],
      "extensions": [
        {"name": "rdkit-representation", "version": 1000, "toolkitVersion": "2018.03.1.dev1",  "aromaticAtoms": [], "aromaticBonds": [], "cipCodes": [[1, "R"]], "atomRings": []},
        {"name": "partial-charges", "version": 1000, "chargeType":"gasteiger", "toolkit":"RDKit", "toolkitVersion": "2018.03.1.dev1", "values": [-0.374, 0.146, -0.087, -0.045, -0.088, -0.047, 0.213, 0.082, 0.061, 0.057, 0.027, 0.027, 0.027]}   
      ]
    }
  ]
}
```
