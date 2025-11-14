# Model Configurator

This document describes the `Configurator` component, which serves as the primary "glue" for the 3D model interaction scene. It is a stateful React component (part of the [overall framework](../Framework.md)) that mounts and manages a `three.js` environment (see [Three.js setup](../../ThreeJS/index.md)).

This component is responsible for:

  * Initializing the 3D renderer, scene, camera, and lighting.
  * Loading and displaying the floor, either as a default rectangle or a custom shape from the [Floorplan Configurator](../FloorplanConfigurator.md).
  * Instantiating and managing the [procedural `WrappedCrate` model](./WrappingLogic.md).
  * Handling all user interactions from the `ConfiguratorSidebar`.
  * Running all [collision detection logic](./Collision.md) to ensure the model stays within the floor bounds.

-----

## Core Concepts

The `Configurator` works by blending React's state and component lifecycle with the imperative `three.js` library.

### 1\. React + Three.js Integration

  * **`useRef` for 3D Objects:** All `three.js` objects that need to be persisted (like the `scene`, `renderer`, `modelRef`, `floorRef`, etc.) are stored in React `useRef` hooks. This gives us a stable reference that doesn't trigger re-renders.
  * **`useState` for UI Controls:** All values controlled by the UI (like `modelPosition`, `modelRotation`, `cratePlanks`) are stored in React `useState` hooks.
  * **`useEffect` for Setup/Cleanup:** A single, large `useEffect` hook initializes the entire `three.js` scene on mount. The `return` function from this hook is critical for cleanup, as it properly disposes of all `three.js` objects to prevent memory leaks.

### 2\. Dynamic Floor Generation

The component has two modes for generating the floor based on the `floorplanPoints` prop:

  * **Default Mode (Rectangular):** If `floorplanPoints` is empty, it creates a simple `THREE.PlaneGeometry` and a matching 4-point collision polygon.
  * **Custom Mode (From Floorplan):** If `floorplanPoints` is provided (from the [Floorplan Configurator](../FloorplanConfigurator.md)), it creates a `THREE.ShapeGeometry` from the points. It also generates a precise 2D collision polygon (`floorPolygonRef.current`) from these points, which is then used by the [collision system](./Collision.md).

### 3\. Procedural Model & Footprint

The main model is a live instance of the `WrappedCrate` class, as documented in [WrappingLogic.md](./WrappingLogic.md).

  * **Model:** `modelRef.current` is the `WrappedCrate` instance. Its size is controlled by calling `modelRef.current.setSize(newSize)`.
  * **Footprint:** `footprintRef.current` is a simple `THREE.PlaneGeometry` that is created as a **child of the model**.
      * Its dimensions are calculated to match the crate's "snapped" size.
      * Because it's a child, it automatically inherits the model's position and rotation.
      * This mesh is passed directly to the collision functions.

> **Note:** The footprint used here is a simple `PlaneGeometry` for performance. For a more advanced method of generating a footprint from a *dynamic* model, see the [Silhouette Extraction](./SilhouetteExtractor.md) process.

### 4\. Advanced Collision & Interaction Logic

The component's main feature is its "smart" handlers that prevent invalid user states.

1.  **The Check:** All handlers use the `checkCollision()` function, which in turn calls `isSilhouetteInsideFloorspace` from the [Collision Logic](./Collision.md).
2.  **Position Handling (Bisection):** When a user drags a position slider, if an invalid position is detected, it performs a **binary search** between the last good position and the new invalid one. This allows the model to slide smoothly along walls.
3.  **Rotation Handling (Nudging):** When a user rotates the model, if a collision occurs, the component tries to "nudge" the model towards the floor's center. If it finds a valid spot, it accepts both the rotation and the new nudged position.
4.  **Resize Handling (Pushing):** When a user resizes the crate, if the new size causes a collision, the component attempts to "push" the model away from the growing side (e.g., push on the +X axis if `numPlanksX` increases). If this push is valid, the resize is accepted. If not, it's reverted.

-----

## Key Functions & Handlers

### `checkCollision()`

  * **Type:** `useCallback`
  * **Purpose:** The central collision-checking function.
  * **Logic:**
    1.  Gets the `footprintRef` (the model's silhouette mesh) and `floorPolygonRef` (the 2D floor polygon).
    2.  Calls `isSilhouetteInsideFloorspace(footprint, floorPolygon)`.
    3.  Updates the `footprintRef` material's color to **green** (valid) or **red** (invalid).
    4.  Returns `true` or `false`.

### `handlePositionXChange(value)` / `handlePositionZChange(value)`

  * **Type:** `useCallback`
  * **Purpose:** Handles user input for changing the model's X or Z position.
  * **Logic:** Performs the "Bisection" logic described above to find the closest valid position to the wall.

### `handleRotationYChange(value)`

  * **Type:** `useCallback`
  * **Purpose:** Handles user input for changing the model's Y-axis rotation.
  * **Logic:** Performs the "Nudging" logic described above to find a valid position after rotating.

### `handlePlankChange(axis, value)`

  * **Type:** `useCallback`
  * **Purpose:** Handles user input for resizing the crate.
  * **Logic:** Performs the "Pushing" logic described above to validate a resize. Also re-builds the `footprintRef.current.geometry` to match the new crate size.