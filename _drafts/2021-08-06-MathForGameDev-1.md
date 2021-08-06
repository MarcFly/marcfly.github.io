---
title: "Math for GameDev - Part 1: Vectors & Dot product"
description: Notes of Freya Holmer's "Math for GameDev" series.
categories:
 - Math
 - Games
tags:
 - math
 - game development
 - 
last_modified_at: 2021-08-06T11:00:00-01:00
---

[Freya Holmer](https://twitter.com/FreyaHolmer) published some time ago some classes on the basic maths required for gamedev. Having myself not been a great math student in university I think it is important for myself to have notes accessible in a form I am comfortable, as well as getting class from an incredible content creator and programmer. Check her out please!

# Vectors Basics

 - `length` : **absolute** magnitude or scalar size of a vector. Calculating the magnitude of a vector varies by dimentions, explained later.
 - `absolute` : refering to the positive part of numbers, also known as unsigned.
 - `direction` : unit representation of a vector
 - `unit / normalized vector` : vector which its length is 1. Basically `v / length(v) = (vx / length(v), vy / length(v), ...)`
 - `sign(v)` : general representation of the direction of a vector. It should return a vector according to the vector dimention input, normalized (`length = 1`).
    - A `0 direction` vector, can be interpreted in a variety of waysly. Generally we want 1 in programming
 - `distance` : length of the vector that joins two positions : `abs(length(vB - vA))`
 - `signed distance` : usually we use the positive term of a distance, sometimes we want also a negative want, thuse we don't get the absolute : `length(vB - vA)

When using more dimentions you will want to use different colors per dimention. Usually, we use **<span style="background:white"><span style="color:red">R</span><span style="color:green">G</span><span style="color:blue">B</span></span>** color space to map to **<span style="background:white"><span style="color:red">X</span><span style="color:green">Y</span><span style="color:blue">Z</span></span>** axis.

Adding new dimentions in an orthographic space, each new axis will be perpendicular ot the previous ones.

Vectors really define a point in space from `0o` (Origin point).

# 2D Vectors - `vector2D(x, y)`

**Addition:** `v1 + v2 = (v1x + v2x, v1y + v2y)` - **Commutative** -> `v1 + v2 = v2 + v1`

**Substraction:** `v1 - v2 = (v1x - v2x, v1y - v2y)` - **NON Commutative** -> `v1 - v2 != v2 - v1`
 - Why not commutative? -> We are changing the sign of v2 (negate components), then perform an addition of the vectors.
 - Ex: `(2,1) - (1,1) = (1, 0)` but `(1,1) - (2,1) = (-1,0)`

**Multiplication:** There are different methods of multiplying and dividing vectors. In most cases, the most straightforward way of multipying the components against the components of another vector is not widely used. Multiplication normally refers to a scalar multiplying a vector `v1 * scalar = (v1x * scalar, v1y * scalar)`. Other caseswill be shown later.

Vector interpretation depends on its purpose, by themselves vectors don't define anything but a point. Be clear about what a vector is supposed to be interpreted as (direction, translation, point, worldspace, localspace,...).

 - `sign(vector2D(x,y))` : new we will be getting a `vector2D()` in its unit form : `v1 / length(v1)`

## Why are Normalized vectors useful?

When we wan to do something along a direction, we don't want to add further changes than intended. A unit / normalized vector provides just a direction and then we can modify such direction to to fit your magnitude needs.

## Length of a vectorin 2D (and 3D)

To normalize a vector we need the length of it. In 1D, the main component is already the scalar part of a vector, but in higher dimentions we need other things. We just use basic trigonometry, the Pythagorean Theorem:

~~~c++
vector2D v
v_length = sqrt(v.x ^ 2 + v.y ^ 2)

vector3D u
u_length = sqrt(u.x ^ 2 + u.y ^ 2 + u.z ^ 2)
~~~

In higher dimentions, I really don't know at this point, we'll figure it out.

## Distance between 2 Vectors = length(v2 - v1)

As stated in the beginning, the distance is the length of a vector that joins two points. A vector defines a point, thus a substraction between point2 and point1 will define a vector from point1 to point2. Then we have to get the length of the newly created vector:

~~~c++
vector2D v,u
distance_v_u = length( vector2D( u.x - v.x, u.y - v.y ) )

distance_fun( v , u )

// Same for 3D, but adding z component
~~~

## Example: Point along a direction

To get a point along a direction represented by an origin point `o` and an end point `v`:
 - Calculate vector `u` that joins those points
 - Get the normalized version of `u`, `u_norm`
 - Multiply `u_norm` to know where to get a point `u_point`
 - Add `o` to `u_point`, to get final point `u_final`
 - Congrats you have got a point between `0o` and `v`

~~~c++
vector2D o,v
vector2D u = v - o
float length_u = sqrt(u.x ^ 2 + u.y ^ 2)
vector2D u_norm = u / length_u
float scalar_value = RandomValue()
vector2D u_point = u_norm * scalar_value
vector2D u_final = u_point + o
~~~

# Dot Product `dot(v1, v2) = v1 x v2`

The most starightforward way of understanding a dot product is that we are projecting a vector over another. Imagine a light that shines perpendicular to `v1`, `v2` will cast a shadow over `v1` that will result in another vector that is a smaller version of `v1`.

The result of the operation is a scalar value that indicates the percentage of `v1` that is obfuscated by `v2`.

~~~c++
vector2D v,u
dot_u_v = v.x * u.x + v.y*u.y
~~~

## Use Case 1: Squared Unsigned Distance of a Vector

If we run that operation by making sure that `u = v`, we get the same operation as the length without computing the squared root of it. In some cases, computing the square root is more costly that squaring up the value we want to compare!

## Use Case 2: Direction difference between 2 vectors

What makes this operation extremely useful is that when we `normalize` the vectors and perform the operation, we are returned the cosine of the angle bewteen the vectors!

This ends up helping us by using directly the value as a really fast indication of direction between those vectors.

Further down the line we can obtain the `angle` between these two vectors in order to get detail information on direction, but taht will come in another post!