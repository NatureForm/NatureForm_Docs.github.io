# 2D Floorplan Collision & Utilities

This document outlines the utility functions used for **2D collision and containment checks**. The primary goal of this code is to determine if a 3D object's 2D footprint (its "silhouette") is fully contained within a 2D floorplan polygon.

This functionality is essential for interactive configurators to validate object placement, preventing users from moving items outside a defined room, placing objects on top of each other, or having them intersect with walls.

---

## Core Logic: `isSilhouetteInsideFloorspace`

The main function, `isSilhouetteInsideFloorspace`, performs a robust, two-part check to confirm an object is fully contained within the floorplan.

For an object to be considered "inside," **both** of the following conditions must be true:

1.  **All Points Inside:** Every single vertex of the object's 2D silhouette must be *inside* the floorplan polygon. This check is performed by the `isPointInPolygon` helper.
2.  **No Edges Intersect:** No edge (the line between two vertices) of the silhouette can *cross* any edge of the floorplan polygon. This check is performed by the `doLinesIntersect` helper.

This dual-check system is crucial. It correctly handles complex cases, such as a large "U" shaped object attempting to wrap around a small part of the floorplan, which might have its vertices inside but its edges intersecting.

---

## Function Reference

### `isSilhouetteInsideFloorspace(silhouetteMesh, floorPolygon)`

* **Purpose:** The primary function. Checks if a given silhouette mesh is fully contained within a floorplan polygon.
* **Parameters:**
    * `silhouetteMesh`: The `THREE.Mesh` representing the object's 2D footprint.
    * `floorPolygon`: An array of 2D points (e.g., `{x: 10, y: 5}`) defining the floor's boundary.
* **Returns:** `true` if the silhouette is fully inside, `false` otherwise.

### `getTransformedSilhouetteVertices(silhouetteMesh)`

* **Purpose:** Extracts a 2D vertex list (a polygon) from a 3D silhouette mesh.
* **Process:**
    1.  Ensures the mesh's world matrix is up-to-date.
    2.  Reads the vertex `position` data from the mesh's geometry.
    3.  Applies the mesh's `matrixWorld` (its final world position, rotation, and scale) to every vertex.
    4.  Converts the 3D `(x, y, z)` coordinates into 2D `(x, y)` coordinates by **using the `x` and `z` values** (the top-down footprint).
    5.  De-duplicates the vertices to create a clean polygon.
* **Returns:** An array of unique 2D points (e.g., `[{x: 1, y: 1}, {x: 2, y: 1}, ...]`).

### `doLinesIntersect(p1, p2, p3, p4)`

* **Purpose:** A mathematical helper to check if two 2D line segments, (p1, p2) and (p3, p4), intersect.
* **Algorithm:** Uses a standard computational geometry algorithm based on the **orientation** (clockwise, counter-clockwise, or collinear) of the three-point combinations.
* **Returns:** `true` if the segments cross, `false` otherwise.

### `isPointInPolygon(point, polygon)`

* **Purpose:** A mathematical helper to check if a 2D point is inside a 2D polygon.
* **Algorithm:** Implements the **Ray-Casting Algorithm**. It "casts" an imaginary ray from the point in one direction (e.g., positive X) and counts how many times it crosses the polygon's edges.
    * An **odd** number of crossings means the point is **inside**.
    * An **even** number of crossings means the point is **outside**.
* **Returns:** `true` if the point is inside, `false` otherwise.