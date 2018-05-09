# CommonChem

CommonChem is a data format for chemical information.

Here is a simple example:

```json
{
  "commonchem": 10,
  "molecules": [
    {
      "name": "ethane",
      "atoms": [
        {"z": 6, "impHs": 3},
        {"z": 6, "impHs": 3}
      ],
      "bonds": [
        {"type": 1, "atoms": [0, 1]}
      ]
    }
  ]
}
```

Find more example files in the [examples](examples) directory.

## Specification

The core specification is in the file [spec.md](spec.md).

## JSON Schema

The [schema](schema) directory contains a [JSON Schema](http://json-schema.org) for CommonChem.

## Extensions

There are a number of extension specifications in the [extensions](extensions) directory.

## Implementations

- RDKit has beta support as of the 2018.03.1 release. Find it under [MolInterchange](https://github.com/rdkit/rdkit/tree/master/Code/GraphMol/MolInterchange).
- Open Babel support is planned.
