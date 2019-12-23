# VRMC_vrm

## Contents

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Contributors](#contributors)
- [Status](#status)
- [Dependencies](#dependencies)
- [Overview](#overview)
  - [Format and Extension](#format-and-extension)
  - [Version](#version)
  - [Model's Meta Information](#models-meta-information)
  - [Humanoid](#humanoid)
  - [Model Normalization](#model-normalization)
  - [BlendShape](#blendshape)
  - [First Person](#first-person)
  - [LookAt](#lookat)
  - [SpringBone](#springbone)
  - [Material](#material)
  - [Constraint](#constraint)
  - [Update Order](#update-order)
- [glTF Schema Updates](#gltf-schema-updates)
  - [Coordinate Units](#coordinate-units)
  - [Unused Items](#unused-items)
  - [Naming Restrictions](#naming-restrictions)
  - [Mesh Storage Restrictions](#mesh-storage-restrictions)
- [JSON Schema](#json-schema)
- [Error Handling](#Error-Handling)
- [Known Implementations](#known-implementations)
  - [VRM-0.0](#vrm-00)
- [Resources](#resources)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Contributors

* Shindo Tatsuro
* Hirose Junmichi
* Po-Chang Su

## Status

* In development

## Dependencies

Written against the glTF 2.0 spec.

* Require VRMC_materials_mtoon extension
* Require KHR_materials_unlit extension
* Require KHR_texture_transform extension

## Overview

Store the information of the humanoid model for VR avatar in the GLTF scene.
By adding more information in the VRM extension along with additional constraints imposing on existing parts in GLTF, humanoid models can be manipulated in unified manners from the program.

### Format and Extension

Save in `.glb` format and use `.vrm` as the extension.

### Version

`extensions.VRMC_vrm`

| Name        | Note                      |
|:------------|:--------------------------|
| specVersion | VRM specification version |

The exporter implementation version is exported to `assets.generator`.

### Model's Meta Information

`extensions.VRMC_vrm.meta`

#### Model name, author, etc.

| Name               | Note                                                                                                                           |
|:-------------------|:-------------------------------------------------------------------------------------------------------------------------------|
| title              | Describe the name of the avatar model                                                                                          |
| version            | Describe the version that creates the model                                                                                    |
| author             | Describe the name of the author of the model                                                                                   |
| contactInformation | Describe the contact information of the author                                                                                 |
| reference          | Describe original / related works of the avatar (URL), if any                                                                  |
| thumbnailImage     | The index to access the thumbnail image of the avatar model in gltf.images. The texture resolution of 2048x2048 is recommended |

#### Personation / Characterization Permission
##### A person who can perform with this avatar

`extensions.VRMC_vrm.meta.allowedUser`

| Name                     | Note |
|:-------------------------|:-----|
| OnlyAuthor               |      |
| ExplicitlyLicensedPerson |      |
| Everyone                 |      |

##### Perform violent acts with this avatar

`extensions.VRMC_vrm.meta.violentUsage`

| Name     | Note |
|:---------|:-----|
| Disallow |      |
| Allow    |      |

##### Perform sexual acts with this avatar

`extensions.VRMC_vrm.meta.sexualUsage`

| Name     | Note |
|:---------|:-----|
| Disallow |      |
| Allow    |      |

##### Commercial use

`extensions.VRMC_vrm.meta.commercialUsage`

| Name     | Note |
|:---------|:-----|
| Disallow |      |
| Allow    |      |

##### extensions.VRMC_vrm.meta.otherPermissionUrl

* Describe the URL links of license with regard to other permissions

#### Redistribution / Modifications License

##### License Type

`extensions.VRMC_vrm.meta.license`

| Name                      | Note                                               |
|:--------------------------|:---------------------------------------------------|
| Redistribution Prohibited | Redistribution Prohibited                          |
| CC0                       | Copyright Wavier (CC0)                             |
| CC_BY                     | Creative Commons CC BY License (CC_BY)             |
| CC_BY_NC                  | Creative Commons CC BY NC License (CC_BY_NC)       |
| CC_BY_SA                  | Creative Commons CC BY SA License (CC_BY_SA)       |
| CC_BY_NC_SA               | Creative Commons CC BY NC SA License (CC_BY_NC_SA) |
| CC_BY_ND                  | Creative Commons CC BY ND License (CC_BY_ND)       |
| CC_BY_NC_ND               | Creative Commons CC BY NC ND License (CC_BY_NC_ND) |
| Other                     | Other                                              |

##### extensions.VRMC_vrm.meta.otherLicenseUrl

Describe the URL links of other license.

### Humanoid

`extensions.VRMC_vrm.humanoid`

VRM defines specifications of the humanoid model.
Here the section which describes humanoid bones in the GLTF node hierarchy is called Humanoid Skeleton.

#### Humanoid Skeleton Specification

* Each bone is unique
* All required bones are included
* Inserting a node that is not related to humanoid bone is allowable (e.g. LowerLeg’s parent is an object cube and this cube’s parent is UpperLeg, etc.).
* `Orientation` is the recommended positional relationship for TPose. It is not recommended that the same (or near the same) coordinate applies to parent and child as it is likely to cause troubles when judging bone orientations in the application. Please set a valid distance (in floating point) that can separate them. 

#### Humanoid Bone (enum)

##### Bone's Bending Direction

To calculate the bone's bending direction, a fixed axis is added to T-Pose.
It is recommended to implement the T-Pose function with taking fixed axis usage into account (optional).

Axis recovery example after import:

```cs
// Calculation method
var forward = (tail - head).normalized; // Bone's forward direction
var axis = new Vector3(1, 0, 0); // The example of spine: x axis is fixed (Fixed Axis column in the table below)
var cross = Vector3.cross(forward, axis); // Get the orthogonal vector from the cross product

// The order of (forward, axis and cross) assigning to (x+-, y+-, z+-) is determined by the system
var y = forward;
var x = axis;
var z = cross;
```

##### Table Header

| Column             | Note                                                                                                                                                                                            |
|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Required           | Without the required bone, the Humanoid cannot be formed, resulting in the error                                                                                                                |
| Parent Bone        | The first bone recognized as the parent bone on top of the current bone during the node traversal in the hierarchy                                                                              |
| Relative Position  | The direction of relative position from the parent bone at T-Pose. It doesn't need to be straight. If the bone's position is the same as (or very close to) the parent bone, an error may occur |
| Fixed Axis         | Align one of the bone's axes during T-Pose, which is able to constraint the bending direction of the joint after import                                                                         |
| Estimated Position | The roughly estimation position of the bone                                                                                                                                                     |

`(tail)` in `required` column means the last connected bone (not required). It is used as a hint when creating a gizmo-like GUI for the model.

It is not relative to the case when `FK` motion is applied.

##### Torso (enum)

| Bone Name  | Required | Parent Bone | Relative Position | Fixed Axis | Estimated Position | Note                                                                  |
|:-----------|:---------|:------------|:------------------|------------|--------------------|-----------------------------------------------------------------------|
| hips       | Required |             | (No parent bone)  | World;X+   |Crotch              | Usually only this bone moves and the other bones in Torso rotate only |
| spine      | Required | hips        | World;Y+          | World;X+   |Top of pelvis       |                                                                       |
| chest      |          | spine       | World;Y+          | World;X+   |Bottom of rib cage  | 0.X is required                                                       |
| upperChest |          | chest       | World;Y+          | World;X+   |                    |                                                                       |
| neck       |          | upperChest  | World;Y+          | World;X+   |Base of neck        | 0.X is required                                                       |

* The corresponding bone of pelvis towards `Y-` direction from hips is deprecated (inverted pelvis)

##### Head (enum)

| Bone Name | Required | Parent Bone | Relative Position | Fixed Axis | Estimated Position | Note                                         |
|:----------|:---------|:------------|-------------------|------------|--------------------|----------------------------------------------|
| head      | Required | neck        | World;Y+          |  World;X+  | Top of neck        |                                              |
| headTail  | (tail)   | head        | World;Y+          | (tail)     | Top of head        |                                              |
| leftEye   |          | head        | N/A               |  N/A       |                    | The model's eye movement controlled by bones |
| rightEye  |          | head        | N/A               |  N/A       |                    | The model's eye movement controlled by bones |
| jaw       |          | head        | N/A               |  N/A       |                    |                                              |

##### Leg (enum)

| Bone Name     | Required | Parent Bone   | Relative Position | Fixed Axis | Estimated Position | Note |
|:--------------|:---------|:--------------|-------------------|------------|--------------------|------|
| leftUpperLeg  | Required | hips          | World;X+          | World;X-   | Groin              |      |
| leftLowerLeg  | Required | leftUpperLeg  | World;Y-          | World;X-   | Knee               |      |
| leftFoot      | Required | leftLowerLeg  | World;Y-          | World;X-   | Ankle              |      |
| leftToes      |          | leftFoot      | World;Y-Z+        | World;X-   |                    |      |
| leftToesTail  | (tail)   | leftToes      | World;Z+          | (tail)     | Toe                |      |
| rightUpperLeg | Required | hips          | World;X-          | World;X-   | Groin              |      |
| rightLowerLeg | Required | rightUpperLeg | World;Y-          | World;X-   | Knee               |      |
| rightFoot     | Required | rightLowerLeg | World;Y-          | World;X-   | Ankle              |      |
| rightToes     |          | rightFoot     | World;Y-Z+        | World;X-   |                    |      |
| leftToesTail  | (tail)   | rightToes     | World;Z+          | (tail)     | Toe                |      |

##### Arm (enum)

| Bone Name     | Required | Parent Bone   | Relative Position | Fixed Axis | Estimated Position | Note |
|:--------------|:---------|:--------------|-------------------|------------|--------------------|------|
| leftShoulder  |          | chest         | World;X-          | World;Y+   |                    |      |
| leftUpperArm  | Required | leftShoulder  | World;X-          | World;Y+   | Base of upper arm  |      |
| leftLowerArm  | Required | leftUpperArm  | World;X-          | World;Y+   | Elbow              |      |
| leftHand      | Required | leftLowerArm  | World;X-          | World;Z-   | Wrist              |      |
| rightShoulder |          | chest         | World;X+          | World;Y+   |                    |      |
| rightUpperArm | Required | rightShoulder | World;X+          | World;Y+   | Base of upper arm  |      |
| rightLowerArm | Required | rightUpperArm | World;X+          | World;Y+   | Elbow              |      |
| rightHand     | Required | rightLowerArm | World;X+          | World;Z+   | Wrist              |      |

##### Finger (enum)

TODO: Thumb's specifications

| Bone Name               | Required    | Parent Bone             | Relative Position       | Fixed Axis          | Estimated Position | Note |
|:------------------------|:------------|:------------------------|-------------------------|---------------------|--------------------|------|
| leftThumbProximal       |             | leftHand                | World;+X+Z              | World;Y+(tentative) |                    |      |
| leftThumbIntermediate   |             | leftThumbProximal       | World;+X+Z              | World;Y+(tentative) |                    |      |
| leftThumbDistal         |             | leftThumbIntermediate   | World;+X+Z              | World;Y+(tentative) |                    |      |
| leftThumbTail           | (tail)      | leftThumbDistal         | World;+X+Z              | (tail)              |                    |      |
| leftIndexProximal       |             | leftHand                | World;+X                | World;Z+            |                    |      |
| leftIndexIntermediate   |             | leftIndexProximal       | World;+X                | World;Z+            |                    |      |
| leftIndexDistal         |             | leftIndexIntermediate   | World;+X                | World;Z+            |                    |      |
| leftIndexTail           | (tail)      | leftIndexDistal         | World;+X                | (tail)              |                    |      |
| leftMiddleProximal      |             | leftHand                | World;+X                | World;Z+            |                    |      |
| leftMiddleIntermediate  |             | leftMiddleProximal      | World;+X                | World;Z+            |                    |      |
| leftMiddleDistal        |             | leftMiddleIntermediate  | World;+X                | World;Z+            |                    |      |
| leftMiddleTail          | (tail)      | leftMiddleDistal        | World;+X                | (tail)              |                    |      |
| leftRingProximal        |             | leftHand                | World;+X                | World;Z+            |                    |      |
| leftRingIntermediate    |             | leftRingProximal        | World;+X                | World;Z+            |                    |      |
| leftRingDistal          |             | leftRingIntermediate    | World;+X                | World;Z+            |                    |      |
| leftRingTail            | (tail)      | leftRingDistal          | World;+X                | (tail)              |                    |      |
| leftLittleProximal      |             | leftHand                | World;+X                | World;Z+            |                    |      |
| leftLittleIntermediate  |             | leftLittleProximal      | World;+X                | World;Z+            |                    |      |
| leftLittleDistal        |             | leftLittleIntermediate  | World;+X                | World;Z+            |                    |      |
| leftLittleTail          | (tail)      | leftLittleDistal        | World;+X                | (tail)              |                    |      |
| rightThumbProximal      |             | rightHand               | World;-X+Z              | World;Y-(tentative) |                    |      |
| rightThumbIntermediate  |             | rightThumbProximal      | World;-X+Z              | World;Y-(tentative) |                    |      |
| rightThumbDistal        |             | rightThumbIntermediate  | World;-X+Z              | World;Y-(tentative) |                    |      |
| rightThumbTail          | (tail)      | rightThumbDistal        | World;-X+Z              | (tail)              |                    |      |
| rightIndexProximal      |             | rightHand               | World;-X                | World;Z-            |                    |      |
| rightIndexIntermediate  |             | rightIndexProximal      | World;-X                | World;Z-            |                    |      |
| rightIndexDistal        |             | rightIndexIntermediate  | World;-X                | World;Z-            |                    |      |
| rightIndexTail          | (tail)      | rightIndexDistal        | World;-X                | (tail)              |                    |      |
| rightMiddleProximal     |             | rightHand               | World;-X                | World;Z-            |                    |      |
| rightMiddleIntermediate |             | rightMiddleProximal     | World;-X                | World;Z-            |                    |      |
| rightMiddleDistal       |             | rightMiddleIntermediate | World;-X                | World;Z-            |                    |      |
| rightMiddleTail         | (tail)      | rightMiddleDistal       | World;-X                | (tail)              |                    |      |
| rightRingProximal       |             | rightHand               | World;-X                | World;Z-            |                    |      |
| rightRingIntermediate   |             | rightRingProximal       | World;-X                | World;Z-            |                    |      |
| rightRingDistal         |             | rightRingIntermediate   | World;-X                | World;Z-            |                    |      |
| rightRingTail           | (tail)      | rightRingDistal         | World;-X                | (tail)              |                    |      |
| rightLittleProximal     |             | rightHand               | World;-X                | World;Z-            |                    |      |
| rightLittleIntermediate |             | rightLittleProximal     | World;-X                | World;Z-            |                    |      |
| rightLittleDistal       |             | rightLittleIntermediate | World;-X                | World;Z-            |                    |      |
| rightLittleTail         | (tail)      | rightLittleTail         | World;-X                | (tail)              |                    |      |

#### TPose Specification

The skeleton must be the T-pose as the initial posture. 
The T-pose specifications are as follows:

* The root bone (topmost bone, hips' ancestor) is at the origin
* The model's heels touch the ground ((Y=0))

| Model's Body Part                                                    | Orientation |
|:---------------------------------------------------------------------|:------------|
| Front                                                                | Z-          |
| Left                                                                 | X+          |
| Up                                                                   | Y+          |
| Torso (hips - spine - chest - upperChest - upperChest - neck - head) | Y+          |
| Left / right leg (upperLeg - lowerLeg)                               | Y-          |
| Left / right foot                                                    | Y-Z-        |
| Left / eight toe                                                     | Z-          |
| Left arm (shoulder - upperArm - lowerArm - hand) and each finger     | X+          |
| Left thumb (Proximal)                                                | X+Z-        |
| Right arm (shoulder - upperArm - lowerArm - hand) and each finger    | X-          |
| Right thumb (Proximal)                                               | X-Z-        |
| Palm                                                                 | Y-          |

### Model Normalization

An algorithm to impose constraints on the GLTF part of the VRM model.

#### Node Normalization

VRM Node is restricted by the followings: 
For initial pose (T-Pose):

* No rotation
* No scaling (1, 1, 1)

#### Mesh Normalization

To achieve node normalization, the mesh needs to be normalized.
The normalized mesh is not skinned and is restricted by the followings:

* Overlapping with initial Humanoid Skeleton (if skin.root exists, add with skin.root's coordinate)
* The forward direction is Z-
* The right direction is X+
* The upward direction is Y+

As a result, `skin.inverseBindMatrices` includes only negative translation of the target bone relative to the world coordinate without rotation and scaling.

```js
// Inverse Matrices
// [x, y, z] = bone.translation - skin.translation
[1, 0, 0, 0]
[0, 1, 0, 0]
[0, 0, 1, 0]
[-x,-y,-z, 1]
```

The restrictions on skinning are as follows:

* No skin.skeleton (scene root) or the node at the origin

#### Normalization Algorithm

* Make the model's pose as T-Pose
* Skin unavailable
  * Multiply each vertex by Node.matrix
* Skin available
  * If there are vertices without BoneWeight, give BoneWeight to skin.root
  *  Skin the mesh and reimport the result as a mesh (bake, freeze)
* Remove the node's rotation / scaling

### BlendShape

VRM extends MorphTarget for humanoids.
BlendShape means grouping multiple MorphTarget (Blink, mouth shape AIUEO, Joy, Angry, Sorrow, Fun).
Also, you can make a blend shape by changing material values (color, texture offset+scale).

#### BlendShapePreset List

##### Facial expression (enum)

| Name    | Note                                                             |
|:--------|:-----------------------------------------------------------------|
| neutral | Standby state. `TODO: Deprecate and provide the ability to bake` |
| joy     |                                                                  |
| angry   |                                                                  |
| sorrow  |                                                                  |
| fun     |                                                                  |

##### Lip-sync

| Name | Note |
|:-----|:-----|
| aa   | あ   |
| ih   | い   |
| ou   | う   |
| ee   | え   |
| oh   | お   |

##### Blink

| Name       | Note                 |
|:-----------|:---------------------|
| blink      | Blink with both eyes |
| blinkLeft  | Blink with left eye  |
| blinkRight | Blink with right eye |

##### BlendShape LookAt

| Name      | Note                                                                                             |
|:----------|:-------------------------------------------------------------------------------------------------|
| lookUp    | For models whose eye gazes move with BlendShape as opposed to bone. More details in [LookAt](#lookat) |
| lookDown  | For models whose eye gazes move with BlendShape as opposed to bone. More details in [LookAt](#lookat) |
| lookLeft  | For models whose eye gazes move with BlendShape as opposed to bone. More details in [LookAt](#lookat) |
| lookRight | For models whose eye gazes move with BlendShape as opposed to bone. More details in [LookAt](#lookat) |

##### User Definition

| Name   | Note                                                                             |
|:-------|:---------------------------------------------------------------------------------|
| custom | Customize the BlendShape for the application. Please use Name for identification |

#### BlendShape Specification

`extensions.VRMC_vrm.blendshape`

| Name                          | Note                                                                                                            |
|:------------------------------|:----------------------------------------------------------------------------------------------------------------|
| blendShapeGroups[*].preset    | Target BlendShapePreset                                                                                         |
| blendShapeGroups[*].name      | Any. (A character string that is unique and can be used as the file name)                                       |
| blendShapeGroups[*].is_binary | In the case of `True`: value!=0 will be considered as 1                                                         |
| blendShapeGroups[*].values    | BlendShapeBind list (described later)                                                                           |
| blendShapeGroups[*].materials | MaterialValueBind list (described later)                                                                        |
| blendShapeGroups[*].ignoreBlink  | When this BlendShape Weight is not 0, the weight of blink, blink_L, blink_R are forced to be 0               |
| blendShapeGroups[*].ignoreLookAt | When this BlendShape Weight is not 0, the weight of lookUp, lookDown, lookLeft, lookRight are forced to be 0 |
| blendShapeGroups[*].ignoreMouth  | When this BlendShape Weight is not 0, the weight of A, I, U, E, O are forced to be 0                         |

##### BlendShapeBind

`extensions.VRMC_vrm.blendshape[*].binds[*]`

Bind BlendShape to MorphTarget.

| Name   | Note                                                                                                               |
|:-------|:-------------------------------------------------------------------------------------------------------------------|
| mesh   | Index of target mesh                                                                                               |
| index  | Index of target morph (All primitives have the same morph [Mesh Storage Restrictions](#mesh-storage-restrictions)) |
| weight | Applied Morph value                                                                                                |

##### MaterialValueBind

Bind BlendShape to MaterialValue 

| Name        | Note                                             |
|:------------|:-------------------------------------------------|
| material    | Index of target material                         |
| target      | Material's items (color, uvScale, uvOffset)      |
| targetValue | Applied material value (float4)                  |


`extensions.VRMC_vrm.blendshape[*].materialValues[*].type`

| Name     | Note                                   |
|:---------|:---------------------------------------|
| color    | Change mainColor                       |
| uvScale  | Change the UV parameter of the texture |
| uvOffset | Change the UV parameter of the texture |

#### BlendShape Update Algorithm

##### BlendShape Identification

* If preset is not custom, use preset for identification
* If preset is custom, use name for identification

##### MorphTarget

* Set all the morph targets to 0
* The value is accumulated based on input blend shapes' weights `void AccumulateValue(BlendShapeClip clip, float value)`
* The accumulated values are all applied at once

##### Material Value

* Set all the materials to their initial states (not 0)
* The value is accumulated based on input blend shapes' weights `void AccumulateValue(BlendShapeClip clip, float value)`
* The accumulated values are all applied at once `Base + (A.Target - Base) * A.Weight + (B.Target - Base) * B.Weight`

### First Person

VRM defines first-person view settings for VR.
lookAt.offsetFromHeadBone can be used as a reference position for the HMD.

#### MeshAnnotation

Control renderings in Mesh units

`extensions.VRMC_vrm.firstPerson.meshAnnotations[*]`

| Name            | Note                  |
|:----------------|:----------------------|
| mesh            | Index for target mesh |
| firstPersonFlag | Described below       |

firstPersonFlag. When using a model in the VR application, the renderings from the HMD and the others are separated.

#### MeshAnnotation (enum)

`thirdPersonOnly` is for hiding the rendering of the model's head portion in VR. Since we assumed that the VR camera's viewpoint is at the head position, by setting thirdPersonOnly, the situation where users may inadvertently see the model's face, head and hair can be prevented.

`firstPersonOnly` is a setting that objects rendered by the first-person camera can only be seen from the first-person view. Normally `firstPersonOnly` will not be used unless the application side needs it for special purposes.

Set `both` for objects that do not necessarily being rendered separately so that those objects can be rendered by all the cameras.

`auto` is used to divide the mesh into the head portion (`thirdPersonOnly`) and the others (`both`). See the details in the next section.

| Name            | First Person Camera (VR viewpoint) | Other Cameras     | Example                                                                      |
|:----------------|:-----------------------------------|:------------------|------------------------------------------------------------------------------|
| thirdPersonOnly | Rendering disable                  | Rendering enable  | Objects that block VR view such as face, eyes, head, hair, hat, helmet, etc. |
| firstPersonOnly | Rendering enable                   | Rendering disable | User interface that is only visible from the user's perspective view         |
| both            | Rendering enable                   | Rendering enable  |                                                                              |

#### MeshAnnotation.Auto Algorithm

* Check all the vertices in Mesh and collect vertices with bone weights for Head and its descendant
* Binarize Mesh into the one containing triangles with the above-mentioned vertices and the other one containing triangles with the rest of the vertices 
* Set the mesh containing the above-mentioned vertices as ThirdPersonOnly, and the mesh containing the other vertices as Both

### LookAt

`extensions.VRMC_vrm.lookAt`

VRM defines eye gaze control for Humanoid.

| Name               | Note                                                                 |
|:-------------------|:---------------------------------------------------------------------|
| lookAtType         | Bone or BlendShape                                                   |
| offsetFromHeadBone | Offset from head bone to reference position of lookAt (between eyes) |
| horizontalInner    | The movable range of horizontal inward direction                     |
| horizontalOuter    | The movable range of horizontal outward direction                    |
| verticalDown       | The movable range of vertical downward direction                     |
| verticalUp         | The movable range of vertical upward direction                       |

#### LookAtType

| Name       | Note                                                                          |
|:-----------|:------------------------------------------------------------------------------|
| bone       | Control the eye gazes with leftEye bone and rightEye bone                     |
| blendShape | Control the eye gazes with BlendShape's LookAt, LookDown, LookLeft, LookRight |

blendShape type can also be set to morph type and UV type (BlendShape setting)

#### Horizontal Inward / Outward, Vertical Upward / Downward

Adjust the movable range for the eyes.

##### Horizontal Inward Direction

`extensions.VRMC_vrm.lookAt.horizontalInner`

* The left eye moves right
* The right eye moves left
* Bone type: outputScale specifies the maximum rotation angle based on the Euler angle (radian) of the leftEye / rightEye bone
* BlendShape type: outputScale specifies the maximum applicable degree of LookLeft / LookRight BlendShape (up to 1.0)

```
Y = clamp(yaw, 0, horizontalInner.inputMaxValue)/horizontalInner.inputMaxValue * horizontalInner.outputScale 
```

##### Horizontal Outward Direction

`extensions.VRMC_vrm.lookAt.horizontalOuter`

* The left eye moves left
* The right eye moves right
* Bone type: outputScale specifies the maximum rotation angle based on the Euler angle (radian) of the leftEye / rightEye bone
* BlendShape type: outputScale specifies the maximum applicable degree of LookLeft / LookRight BlendShape (up to 1.0)

```
Y = clamp(yaw, 0, horizontalOuter.inputMaxValue)/horizontalOuter.inputMaxValue * horizontalOuter.outputScale 
```

##### Vertical Downward Direction

`extensions.VRMC_vrm.lookAt.verticalDown`

* The left eye moves downwards
* The right eye moves downwards
* Bone type: outputScale specifies the maximum rotation angle based on the Euler angle (radian) of the leftEye / rightEye bone
* BlendShape type: outputScale specifies the maximum applicable degree of LookLeft / LookRight BlendShape (up to 1.0)

```
Y = clamp(yaw, 0, verticalDown.inputMaxValue)/verticalDown.inputMaxValue * verticalDown.outputScale 
```

##### Vertical Upward Direction

`extensions.VRMC_vrm.lookAt.verticalUp`

* The left eye moves upwards
* The right eye moves upwards
* Bone type: outputScale specifies the maximum rotation angle based on the Euler angle (radian) of the leftEye / rightEye bone
* BlendShape type: outputScale specifies the maximum applicable degree of LookLeft / LookRight BlendShape (up to 1.0)

```
Y = clamp(yaw, 0, verticalUp.inputMaxValue)/verticalUp.inputMaxValue * verticalUp.outputScale 
```

##### Bone Type

The Yaw and Pitch values (after the movable range adjustment) output as Euler angle are applied to to the LocalRotation of the leftEye and rightEye bones, respectively.

##### BlendShape Type

The Yaw and Pitch values (after the movable range adjustment) output as BlendShape weight are applied to LookLeft, LookRight, LookDown, LookUp BlendShape, respectively.

#### LookAt Algorithm

* The eye gazes are applied to the Yaw/Pitch angle formed by the `HeadBone` (normally it's FirstPersonBone) forward vector and the vector from `HeadBone + firstPersonBoneOffset` to the eye gaze `LookAtObjectPoint`
* x, y = ClampAndScale(yaw, pitch)
* apply(x, y)

##### Bone type

| bone and yaw, pitch                | Note                                           |
|:-----------------------------------|:-----------------------------------------------|
| leftEye + yaw (left)               | Apply horizontalOuter to output as Euler angle |
| leftEye + yaw (right)              | Apply horizontalInner to output as Euler angle |
| rightEye + yaw (left)              | Apply horizontalOuter to output as Euler angle |
| rightEye + yaw (right)             | Apply horizontalInner to output as Euler angle |
| leftEye or rightEye + pitch (down) | Apply verticalDown to output as Euler angle    |
| leftEye or rightEye + pitch (up)   | Apply verticalUp to output as Euler angle      |

##### BlendShape type

| bone and yaw, pitch               | Note                                                          |
|:----------------------------------|:--------------------------------------------------------------|
| leftEye + yaw(left)               | Apply horizontalOuter to output as BlendShape LookLeft value  |
| leftEye + yaw(right)              | Apply horizontalInner to output as BlendShape LookRight value |
| rightEye + yaw(left)              | Apply horizontalOuter to output as BlendShape LookLeft value  |
| rightEye + yaw(right)             | Apply horizontalInner to output as BlendShape LookRight value |
| leftEye or rightEye + pitch(down) | Apply verticalDown to output as BlendShape LookDown value     |
| leftEye or rightEye + pitch(up)   | Apply verticalUp to output as BlendShape LookDown value       |

LookAt BlendShape has MorphTarget type and TextureUVOffset type. The processing here is the same regardless of which type is going to be used.

### SpringBone

The VRM defines the swaying object that does not rely on the physical engine.
It is assumed to be used for the appearance of swaying hair and clothes.

| Name           | Note               |
|:---------------|:-------------------|
| boneGroups     | SpringBone list    |
| colliderGroups | ColliderGroup list |

#### SpringBone

| Name           | Note                                                                         |
|:---------------|:-----------------------------------------------------------------------------|
| comment        | A character string (any)                                                     |
| stiffness      | The force to return to the initial pose                                      |
| gravityPower   | Gravitational acceleration                                                   |
| gravityDir     | The direction of gravity. A gravity other than downward direction also works |
| dragForce      | Air resistance. Deceleration force                                           |
| center         | The index of the node to be the local coordinate                             |
| hitRadius      | The radius of the sphere used for the collision detection with colliders     |
| bones          | The node index array to set SpringBone                                       |
| colliderGroups | Process collision detection. colliderGroup index                             |

#### ColliderGroup

| Name      | Note                                  |
|:----------|:--------------------------------------|
| node      | The index of Node with collider setup |
| colliders | Collider list                         |

#### Collider

* For use by SpringBone only and independent of the Physics system.

| Name      | Note                                                                 |
|:----------|:---------------------------------------------------------------------|
| shapeType | Collider shape (sphere, capsule)                                     |
| offset    | The offset from the collider's node                                  |
| rotation  | The local rotation from the collider's node. Euler Angle (Radians)   |
| size      | Collider radius. {radians} for sphere, {radians, length} for capsule |

#### SpringBone Algorithm

TODO:

```cs
// Calculate the next position with Verlet Integration
var nextTail = currentTail
    + (currentTail - prevTail) * (1.0f - dragForce) // Continue moving from the previous frame (with attenuation)
    + ParentRotation * m_localRotation * m_boneAxis * stiffnessForce // Moving target of child bone by parent rotation
    + external // Translation by external force
    ;
```

### Material

Define Toon Shader

* Require VRMC_materials_mtoon extension

UNLIT is required.

* Require KHR_materials_unlit extension

Texture transform.
There are animation definitions by BlendShape.

* Require KHR_texture_transform extension

### Constraint

TODO:

### Update Order

The followings are the recommended update order:

1. Resolve Humanoid Bone
2. Since the head position has been determined, LookAt can be resolved
  * Bone type
  * BlendShape type
3. Update BlendShape
  * LipSync
  * AutoBlink
  * BlendShape type of LookAt
  * External inputs such as controller
4. Apply BlendShape
5. Resolve constraints
6. Resolve SpringBone

## glTF Schema Updates

### Coordinate Units

Metric units based on glTF [coordinate-system-and-units](https://github.com/KhronosGroup/glTF/tree/master/specification/2.0#coordinate-system-and-units).

### Unused Items

The following items are not used:

* animations
* cameras

### Naming Restrictions

* Make all names unique
* Use characters that can be used for the file name

### Mesh Storage Restrictions

#### TANGENT

* TANGENT information is not saved in the exported VRM model. If NormalMap exists, TANGENT can be obtained via MIKK method

#### MorphTarget

* Normals and tangents are not saved in MorphTarget

#### morphTarget Name

* Morph target names are saved in meshes[*].primitives[*].extras.targetNames

#### VertexBuffer

Given an input mesh, each primitive comprised by different buffers is prohibited

* Each primitive must have the same attributes

Each primitive is not allowed to have independent morph.
It is assumed that morph target is set for mesh as opposed to primitive.

* Each primitive must have the same target
* Each target must have the same attributes

##### Recommendation (shared buffer method)

A single VertexBuffer (accessor) is referenced by multiple primitive.indices.

```
primitives[*].attributes (each primitive use the same accessor)
 +----------------------------------+
 |(0)                               |POSITION
 +----------------------------------+
 +----------------------------------+
 |(1)                               |NORMAL
 +----------------------------------+
 +----------------------------------+
 |(2)                               |TEXCOORD_0
 +----------------------------------+
      ^         ^            ^
      |         |            |
primitives      |            |
  [0].indices   |            |
 +-----------+  |            |
 |(3)        |  |            |
 +-----------+  |            |
              [1].indices    |
             +------------+  |
             |(4)         |  |
             +------------+  |
                           [2].indices
                          +-----------+
                          |(5)        |
                          +-----------+

box: accessor
```

* Since there is only one VertexBuffer, we can force all primitives to have the same attributes
* Same as primitive.targets

##### GLTF Standard (divided buffer method)

```
primitives[*].attributes (each primitive uses a unique accessor)
 +-----------+ +----------+ +----------+
 |(0)        | |(3)       | |(6)       |POSITION
 +-----------+ +----------+ +----------+
 +-----------+ +----------+ +----------+
 |(1)        | |(4)       | |(7)       |NORMAL
 +-----------+ +----------+ +----------+
 +-----------+ +----------+ +----------+
 |(2)        | |(5)       | |(8)       |TEXCOORD_0
 +-----------+ +----------+ +----------+
      ^         ^            ^
      |         |            |
primitives
  [0].indices   [1].indices  [2].indices
 +-----------+ +----------+ +----------+
 |(9)        | |(10)      | |(11)      |indices
 +-----------+ +----------+ +----------+

box: accessor
```

* Make all the primitives have the same attributes
  * Example: `primitive[0]`(POSITION, NORMAL, TEXCOORD_0) may have a different structure from `primitive[1]`(POSITION)
* Same as primitive.targets

```
NG

primitives[*].attributes (each primitive uses a unique accessor)
 +-----------+ +----------+ +----------+
 |(0)        | |(3)       | |(6)       |POSITION
 +-----------+ +----------+ +----------+
 +-----------+ +----------+             
 |(1)        | |(4)       |             NORMAL
 +-----------+ +----------+             
 +-----------+                          
 |(2)        |                          TEXCOORD_0
 +-----------+                          
      ^         ^            ^
      |         |            |
primitives
  [0].indices   [1].indices  [2].indices
 +-----------+ +----------+ +----------+
 |(9)        | |(10)      | |(11)      |indices
 +-----------+ +----------+ +----------+

box: accessor
```

## JSON Schema

```js
{
  "extensionsUsed": {
    "VRMC_vrm"
  },
  "extensions": {
    "VRMC_vrm": {
      // The information of VRM extension
      "specVersion": "1.0"
    }
  },
  // The information of general GLTF-2.0
}
```

* https://github.com/vrm-c/UniVRM/tree/master/specification/1.0/schema

GLTF-2.0 JsonSchema.

* https://github.com/KhronosGroup/glTF/tree/master/specification/2.0/schema

## Error Handling

### Contradiction between skin.inverseBindMatrices and Node Tree
  - The node's translations has priority.

### When Vertex Normal is (0,0,0).
  - WIP

### Unsafe Characters and Strings
 - It may not be a good idea to directly use raw strings of filename, HTML value, etc.. as input for VRM.meta and each name attributes as those raw strings in the user's environment might happen to be control characters, excessively long strings or reserved strings.
 Therefore, escaping potentially dangerous characters and strings is important when importing VRM files.
 
 - [reference issue](https://github.com/vrm-c/vrm-specification/issues/40#issue-530561275)

```cs
#example
static readonly char[] EscapeChars = new char[]
{
    '\\',
    '/',
    ':',
    '*',
    '?',
    '"',
    '<',
    '>',
    '|',
};
public static string EscapeFilePath(this string path)
{
    foreach(var x in EscapeChars)
    {
        path = path.Replace(x, '+');
    }
    return path;
}
```

### Deeply Nested JSON in VRM
- Json parser is able to set a limit on the parsing depth by an implementation. See [rfc8259 sec-9](https://www.rfc-editor.org/rfc/rfc8259.html#section-9) for more details. 
Generally most of the VRM files' nesting depth do not exceed 20. However, it is possible to make a VRM file with a deeply nested JSON structure, which may cause stack overflow, etc. 
Therefore, we will leave it to users to decide how to handle it. *[[ref issue]](https://github.com/vrm-c/UniVRM/issues/318)

## Known Implementations

https://vrm.dev/en/vrm_applications/

## Resources

* https://vrm.dev/en/
