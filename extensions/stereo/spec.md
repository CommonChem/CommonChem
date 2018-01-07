---
title: Stereo CommonChem Extension
version: 0.0.1
author:
    - name: Matthew Swain
    - email: m.swain@me.com
---

# Stereo CommonChem Extension

**Extension ID**: `stereo`

## Introduction

Enhanced stereochemistry groups and exotic stereochemistry types.

## Molecule Fields

This extension adds the following fields to the [Molecule object](../../spec.md#molecule-object):

| Name            | Type                                        | Description                                         |
|-----------------|---------------------------------------------|-----------------------------------------------------|
| x-stereo-groups | [[StereoGroup Object](#stereogroup-object)] | Molecule stoichiometry number in the reaction.      |

## StereoGroup Object

The following fields are available:

| Name    | Type         | Description                           |
|---------|--------------|---------------------------------------|
| type    | `string`     | `absolute`, `and`, `or`, `unknown`.   |
| centers | [`integers`] | Array of stereocenters in this group. |

## Examples
