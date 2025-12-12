<!--
Copyright 2025-2026 The Khronos Group Inc.
SPDX-License-Identifier: CC-BY-4.0
-->

# KHR_skeleton_bone

## Contributors

- Aaron Franke, Godot Engine.

## Status

Draft.

## Dependencies

Written against the glTF 2.0 spec.

Optionally depends on the `KHR_implicit_shapes` extension for defining shapes.

## Overview

This extension defines additional properties and requirements for nodes used as bones in a skeleton.

When a node has the `KHR_skeleton_bone` extension defined, it is a bone in a skeleton hierarchy, even if it is an empty object, and even if it is not used as a glTF skin joint. When a node is used as a glTF skin joint, it is also a bone in the skeleton hierarchy, even if it does not have the `KHR_skeleton_bone` extension defined on it. The `KHR_skeleton_bone` extension may be added to joint and non-joint nodes to provide additional properties for the bone, such as its length, mass, and shape. Alternatively, an empty `KHR_skeleton_bone` extension can be added to joint and non-joint nodes to explicitly mark them as bones in the skeleton hierarchy even when the bone does not have any additional properties defined.

When a glTF file has `KHR_skeleton_bone` included in the `"extensionsUsed"` array, all bones in the glTF file MUST follow the requirements defined by this extension, including nodes used as glTF skin joints which do not have the `KHR_skeleton_bone` extension defined on them.

All bones form a contiguous hierarchy descendant from the skeleton root node, meaning that each and every bone MUST have a contiguous chain of bone parenting leading up to a skeleton root node; the parent node of a bone MUST either be another bone or the skeleton root node itself. A glTF skin's skeleton root node is defined as either a node referenced by the `"skeleton"` property of a glTF skin, or if a skin does not define `"skeleton"`, the lowest common ancestor node of all joints in the skin is the skeleton root for that skin.

All bone nodes SHOULD NOT have a mesh, camera, or light attached to the same node, and SHOULD NOT have similar extension-defined components attached to the same node, such that the glTF node serves only as a bone, and any non-bone components are be attached to child glTF nodes. This enables the creation of an optimized dedicated bone hierarchy, important for applications where such a concept is separate from the standard node hierarchy, and other non-bone objects can then be attached as child nodes of the bone nodes.

## glTF Schema Updates

The `KHR_skeleton_bone` extension may be added to a node in the glTF file's `"nodes"` array to explicitly mark it as a bone in a skeleton. Its existence alone has semantic meaning, and any of the following properties may also be defined:

| Property   | Type      | Description                                                             | Default            |
| ---------- | --------- | ----------------------------------------------------------------------- | ------------------ |
| **length** | `number`  | The length of the bone in meters in the local +Y direction, if defined. | No defined length. |
| **mass**   | `number`  | The mass of the bone in kilograms, if defined.                          | No defined mass.   |
| **shape**  | `integer` | The index of the shape referenced by this bone, if defined.             | No defined shape.  |

### Length

The `"length"` property is a number that defines the length of the bone in meters. This property is optional, and if not defined, the bone does not have a defined length.

If a bone has the `"length"` property defined, it MUST NOT be negative. The length of the bone is used to determine how far the bone extends along the local Y axis of the bone node. The head of the bone is defined as the node's position, and the tail of the bone is offset from this position by `"length"` in the local +Y direction. The final perceived bone length includes scale, so if the bone node has a `"length"` property set to 0.1 and has a global scale of 3.0 along the local Y axis, the effective global bone length is 0.3 meters.

### Mass

The `"mass"` property is a number that defines the mass of the bone in kilograms. This property is optional, and if not defined, the bone does not have a defined mass.

The mass of the bone MAY be used for physics simulations, such as ragdoll physics, or for other purposes. If an application does not support per-bone masses, it MAY safely ignore this property.

If a bone has the `"mass"` property defined, it MUST NOT be negative. If the mass is positive, it is the mass of the bone in kilograms. If the mass is set to exactly `0.0`, it indicates that the bone is massless, and may be used for purposes such as an explicitly non-physical bone, or may provide a shape for its first ancestor bone that has a non-zero mass. If the mass is not defined, the application MAY choose to infer the mass of the bone using any heuristic it desires.

If all bones in a skeleton have their `"mass"` property defined, and the skeleton has a physics motion parent, the total of all bone masses SHOULD add up to the mass of the physics body defined by the parent node's physics motion properties, within the margin of floating-point error. If some or no bones have their `"mass"` property defined, the application MAY choose to infer the masses of the undefined bones by distributing the remaining mass to them. If the shapes of bones are defined, the volume of each bone's shape is recommended to be the heuristic used for this distribution.

### Shape

The `"shape"` property is an integer index of the shape referenced by this bone. This property is optional, and if not defined, the bone does not specify a shape.

This is a reference to a shape in the `KHR_implicit_shapes` extension's `"shapes"` array. If the `"shape"` property is defined, the `KHR_implicit_shapes` extension MUST also be included in the glTF file's `"extensionsUsed"` array, and the index MUST be a valid index into the `"shapes"` array defined by that extension. If the `"shape"` property is not defined, the bone does not specify a shape, and any shape MAY be inferred by the application using any heuristic it desires. The shape MAY be used for visualization, hit registration, ragdoll physics, or any other use case.

The position of the shape is defined as the center of the bone's `"length"`, meaning that if a bone has a length of `L`, the shape's local origin is placed at an offset of `L / 2` in the local +Y direction from the bone's position. The shape uses the bone's local basis without additional rotations or scaling applied, meaning that the shape's local basis axes align with the bone's local basis axes.

The `"shape"` property depends on the `"length"` property. If `"shape"` is defined, `"length"` MUST also be defined. This is because the shape's local origin is positioned at the midpoint of the bone's length, which is ambiguous if the length is undefined or inferred. Therefore, shape placement is only valid when an explicit `"length"` value is provided. This requirement ensures that the position of a shape can always be determined correctly, without any ambiguity.

### glTF Object Model

The following JSON pointers are defined representing mutable properties defined by this extension, for use with the glTF Object Model including extensions such as `KHR_animation_pointer` and `KHR_interactivity`:

| JSON Pointer                                    | Object Model Type |
| ----------------------------------------------- | ----------------- |
| `/nodes/{}/extensions/KHR_skeleton_bone/length` | `float`           |
| `/nodes/{}/extensions/KHR_skeleton_bone/mass`   | `float`           |

Additionally, the following JSON pointers are defined for read-only properties:

| JSON Pointer                                   | Object Model Type |
| ---------------------------------------------- | ----------------- |
| `/nodes/{}/extensions/KHR_skeleton_bone/shape` | `int`             |

### JSON Schema

See [node.KHR_skeleton_bone.schema.json](schema/node.KHR_skeleton_bone.schema.json) for the bone schema.

## Known Implementations

- None

## Resources

- Blender bone length: https://docs.blender.org/api/current/bpy.types.Bone.html#bpy.types.Bone.length
- OMI_seat: https://github.com/omigroup/gltf-extensions/tree/main/extensions/2.0/OMI_seat
- VRChat PhysBones: https://creators.vrchat.com/common-components/physbones/
- VRMC_springBone: https://github.com/vrm-c/vrm-specification/tree/master/specification/VRMC_springBone-1.0
