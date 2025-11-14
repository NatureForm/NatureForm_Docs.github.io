# Procedural Crate Generation

This document outlines the logic for the `WrappedCrate` class, a dynamic and performant 3D object used to represent a crate or pallet built from individual planks.

The class is designed for scenarios where a crate needs to be **procedurally resized** (e.g., by a user in a configurator) while maintaining a realistic appearance. It achieves high performance by using `THREE.InstancedMesh` to draw all planks in a single draw call, rather than creating hundreds of individual `THREE.Mesh` objects.

---

## Core Logic: Snapping and Rebuilding

The most important concept in this class is the "snapping" logic. A user might request an arbitrary size (e.g., 1.23m), but the crate can only be built from discrete plank widths (e.g., 0.1m).

The `setSize` method handles this automatically.

1.  **Snapping:** When `setSize(requestedSize)` is called, it doesn't immediately use that size. Instead, it calculates the **nearest valid "snapped" size** that can be built using the given `plankWidth` and `gapSize`.
2.  **Optimization:** The class stores its current snapped size. It compares the *new* snapped size to the *old* one. If they are the same (e.g., the user is dragging their mouse but hasn't moved far enough to add/remove a plank), the function does nothing.
3.  **Rebuild:** If the new snapped size is different, it triggers the internal `rebuild()` method. This function recalculates the position and scale for every single plank needed to build the 6 faces of the crate.
4.  **Instancing:** The `rebuild()` method populates the `InstancedMesh` with the matrix data (position, rotation, scale) for every plank. Finally, it updates the `InstancedMesh`'s `count` to render the new set of planks.

This entire process ensures that the crate always looks correct for its parts, and it prevents costly, high-frequency rebuilds while a user is actively resizing the object.

---

## Class Reference

### `WrappedCrate`

This class extends `THREE.Group` and manages the `InstancedMesh` for all planks.

#### `constructor(initialSize, plankWidth, plankThickness, material, maxPlanks)`

* **Purpose:** Creates a new `WrappedCrate` instance.
* **Parameters:**
    * `initialSize`: `THREE.Vector3` - The starting dimensions of the crate.
    * `plankWidth`: `number` - The width of each individual plank.
    * `plankThickness`: `number` - The thickness (height) of each plank.
    * `material`: `THREE.Material` - The material to be shared by all plank instances.
    * `maxPlanks` (Optional): `number` - The maximum number of planks to pre-allocate memory for. Defaults to `5000`.

#### `.setSize(requestedSize)`

* **Purpose:** The main public method for updating the crate's dimensions.
* **Parameters:**
    * `requestedSize`: `THREE.Vector3` - The *desired* target size for the crate.
* **Behavior:** This method triggers the "Snapping and Rebuilding" logic. The crate's visual size will snap to the nearest valid dimension based on the `plankWidth`, and the geometry will only be rebuilt if this snapped size changes.