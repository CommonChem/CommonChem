# CommonChem

CommonChem is a data format for chemical information.

Here is a simple example:

```json
{
  "commonchem": {"version": "0.0.1"},
  "molecules": [
    {
      "name": "ethane",
      "atoms": [
        {"z": 6, "hcount": 3},
        {"z": 6, "hcount": 3}
      ],
      "bonds": [
        {"order": 1, "start": 0, "end": 1}
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
