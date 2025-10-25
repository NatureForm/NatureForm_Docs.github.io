# Floorplan Editor

## Overview

The Floorplan feature is an interactive, two-part tool that allows users to create a custom 2D room layout. It's designed to be simple and intuitive, starting users with a basic template and then providing a powerful editor to refine their shape.

The entire system is built as a **fully vector-based** editor, ensuring that shapes can be scaled and manipulated with perfect precision, free from any pixelation.

## User Workflow

The user experience is broken into two distinct stages:

1.  **1. Shape Selection:** The user is first presented with a grid of common room "primitives" or templates, such as a simple square, a rectangle, or an L-shaped room.

2.  **2. Interactive Editing:** After selecting a template, the user is taken into the **Floorplan Workspace**. This is an infinite canvas where they can pan, zoom, and directly manipulate the shape to match their exact specifications.

Once finished, the user can save their design, which finalizes the shape's vertex data.

## Key Features & Capabilities

The workspace provides a rich set of editing tools:

* **View Navigation:**
    * **Pan:** Users can click and drag the grid background to pan the viewport.
    * **Zoom:** Using the mouse wheel, users can zoom in and out, with the zoom centered on the cursor for intuitive control.

* **Shape Manipulation:**
    * **Move Vertices:** Users can grab any corner (vertex) of the shape and drag it.
    * **Move Edges:** Users can grab any wall (edge) and move it, which repositions the two connected vertices along that wall's normal.
    * **Add Vertices:** By clicking and dragging a special handle at the midpoint of any wall, users can "pull out" a new corner, splitting one wall into two.

* **Precision and Alignment:**
    * **Precise Length Input:** The length of each wall is displayed as a text label. Users can click this label to type in an exact numerical length for that wall.
    * **Grid Snapping:** All drag operations (for vertices, edges, or new points) automatically snap to a background grid, ensuring clean lines and easy alignment.

## Technical Implementation Highlights

Instead of being a pixel-based (raster) editor like MS Paint, the floorplan tool is a **vector graphics editor** built using **SVG** (Scalable Vector Graphics).

* **Vector-Based by Design:** The room's shape is not stored as pixels. It's stored in React state as a simple **array of vertices** (e.g., `[{x: 10, y: 10}, {x: 100, y: 10}, ...]`). The SVG `<polygon>` element renders this data visually.

* **Benefits of SVG:**
    * **Infinite Scalability:** The shape, grid, and handles can be zoomed in on indefinitely without any loss of quality or pixelation.
    * **Object-Based Interaction:** Every part of the shape (the main polygon, the circular vertex handles, the edge midpoint handles) is its own SVG element. This makes it simple to attach specific event listeners (like `onMouseDown`) to each part.

* **State-Driven Rendering:** All interactions are handled by React. When a user drags a vertex, they are simply updating the `points` array in the component's state. React then re-renders the SVG with the new vertex coordinates, making the editing feel instantaneous.

* **Coordinate Space Transformation:** A key piece of the logic is the set of helper functions that translate coordinates. When a user clicks, the editor must translate that **screen-space** (pixel) coordinate into the **local-space** (vector) coordinate of the shape, accounting for the current pan and zoom level. This allows a user's mouse movement to be correctly applied to the shape's data.