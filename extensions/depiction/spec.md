---
title: Depiction CommonChem Extension
version: 1.0
author:
    - name: Matthew Swain
    - email: m.swain@me.com
---

# Depiction CommonChem Extension

**Extension ID**: `depiction`

## Introduction

Depiction settings that only affect appearance.

Allow inserting text boxes, arrows, shapes, etc?

## Container Fields

This extension adds the following fields to the [Container object](../../spec.md#container-object):

| Name        | Type                                   | Description                  |
|-------------|----------------------------------------|------------------------------|
| x-depiction | [Depiction  Object](#depiction-object) | Global depiction properties. |

## Depiction Object

| Name        | Type        | Description                 |
|-------------|-------------|-----------------------------|
| font-family | `string`    |                             |
| font-size   | `string`    |                             |
| font-color  | `string`    |                             |
| font-weight | `string`    |                             |
| bond-color  | `string`    |                             |
| bond-weight | `string`    |                             |

## Molecule Fields

## Examples

```json
{
  "commonchem": {
    "version": "0.0.1",
    "extensions": [{"id": "depiction", "version": 1000}]
  },
  "x-depiction": {

  }
}
```

