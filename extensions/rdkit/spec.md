---
title: RDKit CommonChem Extension
version: 0.0.1
author:
    - name: Matthew Swain
    - email: m.swain@me.com
---

# RDKit CommonChem Extension

**Extension ID**: `rdkit`

## Introduction

The RDKit CommonChem Extension provides the ability to store commonly derived molecule properties in a CommonChem file. This allows for faster file reading because sanitization may be skipped.

## Format

### Molecule Fields

`TODO`

### Atom Fields

This extension adds the following fields to the [Atom object](../../spec.md#atom-object):

| Name             | Type    | Description                                                              |
|------------------|---------|--------------------------------------------------------------------------|
| x-rdkit-aromatic | boolean | Whether this atom is aromatic according to the RDKit aromaticity model   |

### Bond Fields

This extension adds the following fields to the [Bond object](../../spec.md#bond-object):

| Name             | Type    | Description                                                              |
|------------------|---------|--------------------------------------------------------------------------|
| x-rdkit-aromatic | boolean | Whether this bond is aromatic according to the RDKit aromaticity model   |


The RDKit aromaticity model is defined as follows: `TODO`

## Examples

```json
{
  "commonchem": {
    "version": "0.0.1",
    "extensions": [{"id": "rdkit", "version": "0.0.1"}]
  }
  "molecules": [
    
  ]
}
```
