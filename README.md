Vulkan Grass Rendering
==================================

**University of Pennsylvania, CIS 565: GPU Programming and Architecture, Project 5**

* Ziqiu Wang
* Tested on: Windows 11 Home, Intel Core Ultra 9 290HX Plus @ 2.7GHz 32GB, RTX 5080 16GB (Personal Computer)

## Project Overview

This project implements grass simulation and rendering in Vulkan, based on [Responsive Real-Time Grass Rendering for General 3D Scenes](https://www.cg.tuwien.ac.at/research/publications/2017/JAHRMANN-2017-RRTG/) by Jahrmann and Wimmer (2017). Grass blades are modeled as Bezier curves, animated by forces in a compute shader, and culled and tessellated.

## Project Details

### Grass Rendering and Tessellation

Each grass blade is a Bezier curve. Its control points, orientation, height, width, up vector, and stiffness are packed into four `vec4`s, with random blade properties and positions across the ground plane. The blades each enter the graphics pipeline as one tessellation patch. The tessellation evaluation shader uses De Casteljau's algorithm to calculate the blade geometry. 

### Force Simulations

At each frame, a compute shader updates every blade's tip using the combined forces multiplied by a fixed timestep:

- **Gravity:** combines downward environmental gravity with a smaller force along the blade's front direction to make the blade bend.
- **Recovery:** pulls the tip toward its initial upright position. The restoring force scales with displacement and blade stiffness, so stiffer blades resist bending more strongly.
- **Wind:** uses sine and cosine gusts that vary with position and time. Wind direction, wind strength, and wind alignment determine the force from the wind.

After applying the forces, the shader keeps tips on or above the local ground plane, adjusts `v1` according to the bend, and performs rescaling to roughly preserve the blade's length.

### Culling Tests

Three culling tests are used to reduce the number of blades sent for tessellation:

- **Orientation culling:** compares the viewing direction with the blade's width direction and rejects nearly edge-on blades.
- **View-frustum culling:** projects the root, tip, and Bezier midpoint into clip space and rejects a blade when all three points lie outside the view frustum, with a tolerance value added.
- **Distance culling:** groups blades into buckets using the their distances from the camera. Farther buckets retain fewer blades, and blades beyond the maximum distance are simply discarded.

You may toggle the three tests with keys `1`, `2`, and `3` respectively.

## Performance Analysis

### Number of Grass Blades

### Culling Tests

## References

- [Hash Function Prospector](https://github.com/skeeto/hash-prospector): `lowbias32` used in `hashBlade`
