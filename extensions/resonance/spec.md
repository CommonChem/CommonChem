---
title: Resonance CommonChem Extension
version: 1.0
author:
    - name: Matthew Swain
    - email: m.swain@me.com
---

# Resonance CommonChem Extension

**Extension ID**: `resonance`

## Introduction

Simple representation of chemical structures in cheminformatics systems necessitates the use of integer bond orders and integer charges that are localized on single atoms. However, this often leads to imperfect representations of real-word systems.

Resonance group: List of atoms to delocalize net charge across, average bond orders between.
Bonds into the group continue to be to a specific atom, unless otherwise specified that they should be to the entire group.

## Format

### Molecule Fields

This extension adds the following fields to the [Molecule object](../../spec.md#molecule-object):

| Name        | Type                                     | Description                       |
|-------------|------------------------------------------|-----------------------------------|
| x-resonance | [[Resonance Object](#resonance-object)]  | Array of resonance group objects. |

Example of a molecule with resonance groups:

```json
{
  "atoms": [],
  "bonds": [],
  "x-resonance": [
    {"atoms": [0, 1, 2, 3, 4], "bonds": [5]}
  ]
}
```

### Resonance Object

The following fields are available:

| Name    | Type           | Description                                                                              |
|---------|----------------|------------------------------------------------------------------------------------------|
| atoms   | [`integer`] | Array of atom indices for atoms in this resonance group. **Required**.                      |
| bonds   | [`integer`] | Array of bond indices for bonds where one end should be considered to be connected to the entire group. Each specified bond should have one end connected to an atom in the group and one end connected to an atom outside the group. |
| charge  | `boolean`        | Whether total net charge should be delocalized across group. **Default**: true         |
| radical | `boolean`        | Whether radical electrons should be delocalized across group. **Default**: true        |
| orders  | `boolean`        | Whether total sum of all internal bond orders should be averaged across group. **Default**: true |


## Examples


