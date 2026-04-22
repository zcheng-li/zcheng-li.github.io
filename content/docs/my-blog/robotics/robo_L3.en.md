---
title: "L3.Knimatics"
weight: 1
# bookFlatSection: false
bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# L3.Kinematics

Kinematcis describe the **position** and **orientation** of a robot. These depends on the translational and rotational relationship of **all the joints** for a robot. These are called **transformation**.

> kinematcis = position + orientation

> transformation = translation + rotation

Transformation is defined in terms of specific reference frames i.e. coordinate systems in space.

## Transformation

### Pure Translation Transformation

![Pure Translation Transformation](/img/robo/L3_p11.png)

![alt text](/img/robo/L3_p12.png)

### Pure Rotation Transformation

![alt text](/img/robo/L3_p13.png)

* n: normal (local x-axis)
* o: orientation (local y-axis)
* a: approach (local z-axis)

![alt text](/img/robo/L3_p15.png)

![alt text](/img/robo/L3_p16.png)

### Combined Transformations

Combining Transformation involves **premultiplying** the transformation matrices in the order they occur.

For instance, transform point P in frame A to Frame U:

$${}^U P = {}^U T_C \times {}^U T_B \times {}^U T_A \times {}^A P$$

It means move/rotate in the world coordinate system.

Transformations relative to a moving reference frames involve **postmultiplying** the transformation matrics in the order they occur.

It means move/rotate in the object’s own coordinate system

Suggested reading material:
https://zhuanlan.zhihu.com/p/660742175