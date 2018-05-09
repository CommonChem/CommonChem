# CommonChem

CommonChem is a data format for chemical information.

Here is a simple example:

```json
{
  "commonchem": 1000,
  "defaults": {
    "atom": {"stereo": "unspecified", "chg": 0, "nRad": 0, "z": 6, "impHs": 0, "isotope": 0},
    "bond": {"stereoAtoms": [], "stereo": "unspecified", "type": 1}
  },
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

## Specification

The specification is in the file [spec.md](spec.md).

## JSON Schema

The [schema](schema) directory contains a [JSON Schema](http://json-schema.org) for CommonChem.

## Extensions

There are a number of extension specifications in the [extensions](extensions) directory.

## Implementations

TODO: MIT-licensed reference implementations in C++ for RDKit and Open Babel
