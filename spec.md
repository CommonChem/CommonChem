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
- [Extensions](#extensions)
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
- **Extension fields** follow a strict pattern, and are declared in an extension specification.

### Serialization

It should be possible to use CommonChem with any JSON-compatible serialization format. That is, the format must at least support the features of JSON. It may have additional features, but these should not be required by the Core CommonChem Specification or any CommonChem extension specification.

#### JSON

JSON is the primary serialization format that is recommended for most users. All examples in this specification assume JSON format.

JSON example:

```json
{
  "commonchem": {"version": 1000},
  "molecules": [{
    "name": "ethane",
    "atoms": [{"element": 6, "hcount": 3}, {"element": 6, "hcount": 3}],
    "bonds": [{"order": 1, "atoms": [0, 1]}]
  }]
}
```

#### YAML

[YAML](http://yaml.org) is a data serialization format that is designed to be more human-readable than JSON. It offers a superset of JSON features, although none of these additional features are explicitly required by CommonChem.

YAML example:

```yaml
commonchem:
  version: 1000
molecules:
  - name: ethane
    atoms:
      - element: 6
        hcount: 3
      - element: 6
        hcount: 3
    bonds:
      - order: 1
        start: 0
        end: 1
```

#### MessagePack

[MessagePack](https://msgpack.org) is a binary format that is designed to be more efficient than JSON. It is not human readable, but offers smaller file sizes and faster transmission over the Internet.

### Data Types

The data types used by CommonChem match those supported by JSON.

| Type      | Description                                |
|-----------|--------------------------------------------|
| `object`  | Container with fields and values           |
| `array`   | Ordered list of elements                   |
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
   "commonchem": {"version": 1000},
   "molecules": [{"atoms": [{"element": 6, "hcount": 4}]}]
}
```

It can have the following fields:

| Name       | Type                                  | Description                                                    |
|------------|---------------------------------------|----------------------------------------------------------------|
| commonchem | [Metadata Object](#metadata-object)   | Indicates that the file conforms to a specific version of the CommonChem specification and provides other metadata about the file.  **Required**. |
| molecules  | [[Molecule Object](#molecule-object)] | Array of Molecule objects, each of which describes a chemical structure. |

It is recommended that the `commonchem` field is the first field in the file, but this is solely to help with human-readability.

[Extension fields](#extension-fields) may be added to this object.

### Metadata Object

Every CommonChem file must have a single Metadata object as the value for the `commonchem` field in the top level Container.

Example of a Metadata object:

```json
{
  "version": 1000,
  "extensions": [{"id": "rdkit", "version": "0.0.1"}]
}
```


| Name       | Type                                    | Description                                                  |
|------------|-----------------------------------------|--------------------------------------------------------------|
| version    | `integer`                               | The version of this specification that the file conforms to. **Required**. |
| extensions | [[Extension Object](#extension-object)] | Array of CommonChem extensions used in this file.            |

Each [Extension Object](#extension-object) in the `extensions` field must have a unique `id`. A file cannot conform to multiple versions of the same extension.

### Extension Object

An Extension object indicates that fields from a certain extension are used in the file.

| Name       | Type             | Description                                                                         |
|------------|------------------|-------------------------------------------------------------------------------------|
| id         | `string`         | Extension identifier. Must be unique within this array of extensions **Required**.  |
| version    | `string`         | The version of the extension specification that the file conforms to. **Required**. |
| spec       | `string`         | The URL of the extension specification.                                             |
| supported  | `boolean`        | Whether the application that wrote the file fully supported the extension. **Default**: `true` |

Any application that writes files that contain extension fields should include an array of Extension objects in the `commonchem` Metadata object.

Example of an Extension object:

```json
{"id": "rdkit", "version": "0.0.1"}
```

If an application processes a file that contains extensions that it does not support, every effort should be made to preserve the extension data. However, when writing back to a file, the `supported` field should be set to `false` on the corresponding Extension objects. This indicates to any other applications that subsequently process this file that the data in the fields from that extension may not be fully valid.

```json
{"id": "rdkit", "version": "0.0.1", "supported": false}
```

When reading a file of this type, applications should perform validation checks to flag any invalid data, and automatically re-derive it if possible.

For example, consider the situation where one application stores aromaticity information in an extension. If a second application is then used to process a molecule and break a bond in the aromatic ring, it will read and write the aromaticity extension fields unchanged, even though they are no longer valid for the new structure. If the first application reads the file again, the fact that the extension object has the `supported` field set to `false` indicates that the aromaticity information should be regenerated rather than read from the file.

Applications that read and write CommonChem files must support an extension in its entirety, or not at all. It is not allowed to interpret a specific field from an extension but not others. The only circumstances under which an application may write a field from an unsupported extension is when preserving it unmodified from an input.

[Extension fields](#extension-fields) may be added to this object, however extensions should only add fields to their own extension object, not those for other extensions.

### Molecule Object

A Molecule object describes the structure and properties of a collection of atoms. The atoms may be connected by bonds, but this is not required: Multiple disconnected fragments may be contained in a single Molecule object.

Example of a Molecule object:

```json
{
    "id": "CID6324",
    "name": "ethane",
    "atoms": [
      {"element": 6, "hcount": 3},
      {"element": 6, "hcount": 3}
    ],
    "bonds": [
      {"order": 1, "atoms": [0, 1]}
    ]
}
```

The following fields are available:

| Name        | Type                                    | Description                                           |
|-------------|-----------------------------------------|-------------------------------------------------------|
| name        | `string`                                | A name for this Molecule.                             |
| atoms       | [[Atom Object](#atom-object)]           | An array of Atom objects.                             |
| bonds       | [[Bond Object](#bond-object)]           | An array of Bond objects.                             |
| conformers  | [[Conformer Object](#conformer-object)] | An array of Conformer objects.                        |
| absolute    | `boolean`                               | Absolute stereochemistry flag.                        |

[Extension fields](#extension-fields) may be added to this object.

### Atom Object

The following fields are available:

| Name        | Type      | Description                                                                               |
|-------------|-----------|-------------------------------------------------------------------------------------------|
| element     | `integer` | Atomic number. **Required**.                                                              |
| charge      | `number`  | Formal charge. **Default**: 0.                                                            |
| hcount      | `integer` | Count of implicit hydrogens bonded to this atom. Does not include any hydrogen atoms that exist explicitly in the array of atom objects. **Default**: 0. |
| isotope     | `integer` | Atomic mass number. **Default**: Natural abundance mixture.                               |
| radical     | `integer` | Number of unpaired electrons. **Default**: 0.                                             |
| lonepairs   | `integer` | Number of lone pairs. **Default**: 0.                                                     |
| parity      | `integer` | `1` for clockwise or `-1` for anticlockwise.                                              |

In contrast to many other formats, no assumptions are made about allowed valence states, and therefore the hydrogen count must always be specified (if non-zero).

Hydrogens may be represented as actual atom objects in the molecular graph as an alternative to the `hcount` field. This is necessary if:

- Coordinates are provided for the hydrogen (typically in 3-D)
- The hydrogen has a charge or isotope number
- The hydrogen does not have exactly one bond to a non-hydrogen atom

Tetrahedral stereochemistry is specified using the `parity` field. The order of the bonded atom neighbors is given by their relative order in the molecule's `atoms` array. If the atom has a non-zero `hcount` the hydrogen is considered to be the first atom neighbor in the order. Looking towards the chiral center from the first atom neighbor, the remaining neighbors occur in a clockwise (`parity` of `1`) or anti-clockwise (`parity` of `-1`) direction.

Atom coordinates are specified using the [`conformers`](#Conformer) molecule field.

### Bond Object

The following fields are available:

| Name  | Type        | Description                                                             |
|-------|-------------|-------------------------------------------------------------------------|
| order | `integer`   | Bond order. **Required**.                                               |
| atoms | [`integer`] | Indices of atoms in the atoms array involved in this bond. **Required** |
| style | `string`    | Bond style specifier. `dashed`, `wavy`, `dotted`, `arrow`.              |
| type  | `string`    | Bond type specifier. `dative`, `complex`, `hydrogen`, `ionic`.          |

Allowed bond orders are 0, 1, 2, 3. Zero-order bonds do not have any valence contribution, and should be used for things like coordination bonds and hydrogen bonds. Bond orders 1, 2, and 3 correspond to single, double, and triple bonds respectively.

The bond `atoms` array uses the zero-based index of atoms in the molecule's `atoms` array. It must contain exactly two elements.

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
  "commonchem": {"version": 1000},
  "molecules": [
    {
      "atoms": [{"element": 6}, {"element": 6}, {"element": 8}, {"element": 8}],
      "bonds": [{"atoms": [0, 1]}, {"atoms": [1, 2]}, {"atoms": [1, 3], "order": 2}],
      "conformers": [
        {"dim": 2, "coords": [[1.7321,0.0000], [0.8660, 0.5000], [0.8660, 1.5000], [0.0000, 0.0000]]}
      ]
    }
  ]
}
```

## Extensions

### Extension Fields

Custom fields may be added to certain objects, for example Molecules, Atoms, and Bonds.

Extension fields must use the following format:

```json
{
  "x-<extension>-<property>": "<value>"
}
```

Each field consists of the prefix `x-`, followed by the extension `id`, followed by the property name. This enforces a namespace so extension fields do not clash. The field value may be an object itself with further nested fields. These nested fields are not required to follow the extension field name format.

```json
{
  "commonchem": {
    "version": 1000,
    "extensions": [{"id": "rdkit", "version": "0.0.1"}, {"id": "myextension", "version": "0.0.1"}]
  },
  "molecules": [
    {
      "atoms": [{"element": 6, "hcount": 4, "x-rdkit-aromatic": false}],
      "x-myextension-molmeta": {
        "flavor": 2,
        "comment": "test"
      }
    }
  ]
}
```

Applications should emit a warning when reading and writing files that contain extensions that they do not support. When reading a file with unsupported extensions, the extension fields should be read and stored as generic properties on the relevant Molecule, Atom or Bond. No attempt should be made to actually interpret the data from an unsupported extension. When writing back to a CommonChem output file, these generic properties should be translated back into the original extension fields, preserving the original input. The corresponding [Extension object](#extension-object) should also be written to match the version specified in the input, but the `supported` field should be set to `false`, to indicate that the extension fields are potentially invalid.

Applications should emit an error and abort reading a file upon encountering an extension field with no corresponding [Extension object](#extension-object) in the `commonchem` metadata object. Even if the application recognizes the extension name as one it supports, it is ambiguous which version of the extension has been used and therefore the file should not be read.

### Use Cases

Two main use cases are envisioned for extensions: **Application Extensions** and **Shared Extensions**.

#### Application Extensions

Application extensions should only be used for data where it does not make sense for any other application to interpret it. Applications are discouraged from actually interpreting and making use of application-specific extensions written by other applications. They should be read in and written out without modification, by assigning the values as generic properties to the relevant Molecule, Atom, and Bond objects.

The application name should be used for the extension `id`:

```json
{
  "x-rdkit-property": "value"
}
```

Everything in the field name after the extension `id` is flexible and free for the application developer to use as they see fit. For example, if the extension is used by a company for multiple applications, it may be desirable further specify an application-specific level: 

```json
{
  "x-chemaxon-marvin-property": "value"
}
```

A specification should still be published for application extensions, even if no other applications will implement it. This serves to reserve the extension `id` and provide a public record of the extension's existence.

#### Shared Extensions

One of the goals of the CommonChem project is for as many applications as possible to achieve full support for the Core CommonChem Specification. Therefore, data that is unlikely to be supported by the majority of applications is considered outside the scope of the core specification and should instead be defined in a shared extension that a subset of applications may support.

If multiple applications are using application extensions to store the same or similar data, they should ideally be submitted for inclusion in a future version of the Core CommonChem Specification. However, this might not always be feasible if there is disagreement about the schema, semantics, or whether they should be included at all. In these cases, the community may wish to develop a shared extension.

### Review Process

Anyone may create an extension specification. Submission for review through the CommonChem GitHub repository is encouraged but not required. An approved review has the benefit of reserving the extension `id`, should the situation arise where multiple extensions desire the same `id`. It also serves to publicize the extension to application developers, encouraging support in applications.

## Examples

`TODO`

## Appendix A: Definitions?

## Appendix B: Periodic Table Data?

