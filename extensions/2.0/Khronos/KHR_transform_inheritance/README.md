<!--
Copyright 2026 The Khronos Group Inc.
SPDX-License-Identifier: CC-BY-4.0
-->

# KHR_transform_inheritance

## Contributors

- Aaron Franke, Godot Engine.

## Status

Draft.

## Dependencies

Written against the glTF 2.0 spec.

## Overview

This extension defines whether a node's translation, rotation, and scale follow along with its parent nodes.

A node can have the `KHR_transform_inheritance` extension defined to specify any of these boolean properties. If no properties are defined in the extension, the node behaves the same as if the extension was not present. If some or all of the properties are defined and set to `false`, then those components of the node's transform do not follow along with its parent node, and instead follow along with the glTF scene root.

The glTF node's `"translation"`, `"rotation"`, `"scale"`, and `"matrix"` properties are still defined relative to the node's parent node, the same as they are for nodes without this extension. The properties defined by this extension only control how the local and global transforms of the node are affected by changes to the transforms of its parent node and any further ancestor nodes, they do not redefine the meaning of the glTF node's transform properties themselves.

## glTF Schema Updates

The `KHR_transform_inheritance` extension may be added to a node in the glTF file's `"nodes"` array to control whether the node's translation, rotation, and scale follow along with its parent nodes. The extension defines the following properties:

| Property               | Type      | Description                                                                  | Default |
| ---------------------- | --------- | ---------------------------------------------------------------------------- | ------- |
| **inheritRotation**    | `boolean` | If false, the node's rotation does not follow along with its parent node.    | `true`  |
| **inheritScale**       | `boolean` | If false, the node's scale does not follow along with its parent node.       | `true`  |
| **inheritTranslation** | `boolean` | If false, the node's translation does not follow along with its parent node. | `true`  |

### Inherit Rotation

The `"inheritRotation"` property is a boolean that defines whether or not the node's rotation follows along with its parent node's rotation. This property is optional, and if not defined, the default value is `true`.

If `true` or not specified, the node's rotation follows along with its parent glTF node, if it has one. Meaning, if the parent glTF node or any further ancestor glTF node rotates, then this node will have its global transform rotate along with it, keeping the same local rotation relative to its parent node. This is the same as the default behavior of nodes in glTF without this extension.

If `false`, the node's rotation is not affected by the changes to the transforms of the parent glTF node and any further ancestor glTF nodes, and instead follows along with the glTF scene root. Meaning, if the parent glTF node or any further ancestor glTF node rotates, then this node will not have its global transform rotate along with it, and instead will keep the same global rotation relative to the glTF scene root.

### Inherit Scale

The `"inheritScale"` property is a boolean that defines whether or not the node's scale follows along with its parent node's rotation and scale. This property is optional, and if not defined, the default value is `true`.

If `true` or not specified, the node's scale follows along with its parent glTF node, if it has one. Meaning, if the parent glTF node or any further ancestor glTF node rotates or scales, then this node will have its global transform rotate and scale along with it, keeping the same local scale relative to its parent node. This is the same as the default behavior of nodes in glTF without this extension.

If `false`, the node's scale is not affected by the changes to the transforms of the parent glTF node and any further ancestor glTF nodes, and instead follows along with the glTF scene root. Meaning, if the parent glTF node or any further ancestor glTF node rotates or scales, then this node will not have its global transform rotate or scale along with it, and instead will keep the same global scale relative to the glTF scene root.

> **Implementation Note:** Node rotation can affect how the parent's scale affects the child's scale. For example, if the parent node is scaled by 2 on its X axis, the child node is scaled by 3 on its X axis, and the child is rotated by 90 degrees around its Y axis, then the child's global scale will be 2 on its own Z axis. Therefore, it is not sufficient to only multiply scale numbers together component-wise. At a minimum, the entire basis needs to be considered, meaning the 3x3 matrix subset of the full 4x4 transformation matrix.

> **Author Note:** To avoid problematic behavior, such as skewing/shearing arising from the composition of non-uniform scales and rotations, consider using uniform scaling only, with the scale components all set to the same value. This advice applies regardless of whether this extension is used or not.

### Inherit Translation

The `"inheritTranslation"` property is a boolean that defines whether or not the node's translation follows along with its parent node's transform. This property is optional, and if not defined, the default value is `true`.

If `true` or not specified, the node's translation follows along with its parent glTF node, if it has one. Meaning, if the parent glTF node or any further ancestor glTF node transforms, including translating, rotating, or scaling, then this node will have its global transform move along with it, keeping the same local translation relative to its parent node. This is the same as the default behavior of nodes in glTF without this extension.

If `false`, the node's translation is not affected by the changes to the transforms of the parent glTF node and any further ancestor glTF nodes, and instead follows along with the glTF scene root. Meaning, if the parent glTF node or any further ancestor glTF node transforms, including translating, rotating, or scaling, then this node will not have its global translation move along with it, and instead will keep the same global translation relative to the glTF scene root.

> **Implementation Note:** Node translation is affected by the parent's rotation and scale as well as the parent's translation. For example, if the parent node is rotated 90 degrees around its Y axis, has a uniform scale of 2, and its child has a local translation of (0, 0, 1), then the child's global translation will be (2, 0, 0). Therefore, it is not sufficient to only add translation numbers together component-wise. At a minimum, the entire linear transformation of the parent node needs to be considered, meaning the 3x4 matrix subset of the full 4x4 transformation matrix.

### glTF Object Model

The following JSON pointers are defined representing mutable properties defined by this extension, for use with the glTF Object Model including extensions such as `KHR_animation_pointer` and `KHR_interactivity`:

| JSON Pointer                                                        | Object Model Type |
| ------------------------------------------------------------------- | ----------------- |
| `/nodes/{}/extensions/KHR_transform_inheritance/inheritRotation`    | `boolean`         |
| `/nodes/{}/extensions/KHR_transform_inheritance/inheritScale`       | `boolean`         |
| `/nodes/{}/extensions/KHR_transform_inheritance/inheritTranslation` | `boolean`         |

### JSON Schema

See [node.KHR_transform_inheritance.schema.json](schema/node.KHR_transform_inheritance.schema.json) for the schema.

## Known Implementations

- None

## Resources

- Godot `top_level` property: https://docs.godotengine.org/en/stable/classes/class_node3d.html#class-node3d-property-top-level
