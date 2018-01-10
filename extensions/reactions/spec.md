---
title: Reactions CommonChem Extension
version: 1.0
author:
    - name: Matthew Swain
    - email: m.swain@me.com
---

# Reactions CommonChem Extension

**Extension ID**: `reactions`

## Introduction


## Molecule Fields

This extension adds the following fields to the [Molecule object](../../spec.md#molecule-object):

| Name                      | Type      | Description                                         |
|---------------------------|-----------|-----------------------------------------------------|
| x-reactions-stoichiometry | `integer` | Molecule stoichiometry number in the reaction.      |
| x-reactions-role          | `string`  | Reaction role: reactant, product, solvent, catalyst |

## Atom Fields

This extension adds the following fields to the [Atom object](../../spec.md#atom-object):

| Name            | Type      | Description                                                |
|-----------------|-----------|------------------------------------------------------------|
| x-reactions-map | `integer` | Reaction atom mapping number to map reactants to products. |

## Reaction Object



## Examples

```json
{
  "commonchem": {
    "version": 1000,
    "extensions": [{"id": "reactions", "version": 1000}]
  },
  "x-reactions": [
    {
      "reactants": [],
      "agents": [],
      "products": [],
      "reversible": false
    }
  ]
}
```

