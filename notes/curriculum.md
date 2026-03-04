# Software Rasterizer — Exercise Curriculum

## Phase 1: Setup & Window (Week 1)

### Exercise 1.1 — Hello Window
Create a program using SDL2 (or SFML) that opens a 800×600 window, fills the background with a solid color, and closes cleanly when you press Escape. You should write zero pixel data manually yet — just get the window working.

**Concepts:** SDL2 init/quit lifecycle, event loop, window creation.

### Exercise 1.2 — Manual Pixel Buffer
Allocate a raw `uint32_t` array of size `width × height`. Write a function `setPixel(x, y, r, g, b)` that writes a color into that array. Draw a checkerboard pattern by computing pixel positions mathematically. Display this buffer to the screen using SDL's texture/surface blitting.

**Concepts:** row-major memory layout, color packing (`(r << 16) | (g << 8) | b`), CPU-side framebuffer.

### Exercise 1.3 — Main Loop Timing
Add a frame timer. Measure how long each frame takes, compute FPS, and print it to the console. Cap your loop to ~60 FPS using `SDL_Delay` or equivalent. Make the checkerboard colors animate (shift over time) to confirm the loop is running.

**Concepts:** delta time, frame budget, `SDL_GetTicks`.

---

## Phase 2: Math Foundation (Week 1–2)

### Exercise 2.1 — Vector Library
Implement `Vec2`, `Vec3`, `Vec4` structs with: add, subtract, scalar multiply, dot product, cross product (Vec3), length, normalize. Write unit tests (even just assert statements in main) that verify each operation with known values.

**Concepts:** dot/cross product geometry, normalization, homogeneous coordinates (why Vec4 exists).

### Exercise 2.2 — Matrix Library
Implement a `Mat4` (4×4 float matrix) with: identity, matrix-matrix multiply, matrix-vector multiply. Test that `identity × v == v` and that two known matrices multiply correctly.

**Concepts:** row vs. column major, why 4×4 matrices are used for 3D transforms.

### Exercise 2.3 — Transform Matrices
Build functions that return a Mat4 for each transform:
- `makeTranslation(x, y, z)`
- `makeScale(x, y, z)`
- `makeRotationX/Y/Z(angle)`

Test by transforming a known point and verifying the result by hand.

**Concepts:** affine transformations, why translation requires 4D matrices.

### Exercise 2.4 — Projection Matrices
Implement `makePerspective(fovY, aspect, near, far)` and `makeOrthographic(...)`. Apply perspective to a hardcoded 3D point and verify the output looks correct (closer objects should appear larger). Don't render anything yet — just print values.

**Concepts:** perspective divide, frustum, field of view, clip space.

---

## Phase 3: Drawing 2D Primitives (Week 2)

### Exercise 3.1 — Draw a Line
Implement Bresenham's line algorithm: `drawLine(x0, y0, x1, y1, color)` that writes pixels into your framebuffer. Draw a star/asterisk pattern of lines from the center of the screen to verify it works in all octants.

**Concepts:** Bresenham's algorithm, handling all 8 octants, integer arithmetic.

### Exercise 3.2 — Draw a Triangle (Wireframe)
Using your line function, implement `drawTriangleWireframe(v0, v1, v2, color)`. Draw several triangles at hardcoded screen positions.

**Concepts:** triangle as three edges, 2D screen-space coordinates.

### Exercise 3.3 — Fill a Triangle
Implement scanline triangle filling: for each row between the triangle's min/max Y, compute the left and right X boundaries and fill that horizontal span. Handle the flat-top and flat-bottom cases.

**Concepts:** scanline rasterization, edge walking, top-left fill rule.

---

## Phase 4: 3D to 2D Pipeline (Week 3)

### Exercise 4.1 — Project a Cube
Hardcode 8 vertices of a unit cube. Apply a perspective matrix to each vertex, perform the perspective divide (x/w, y/w), and map to screen coordinates. Draw it as wireframe using your line function.

**Concepts:** clip space → NDC → screen space, viewport transform.

### Exercise 4.2 — Model/View/Projection (MVP)
Add a model matrix (translate/rotate the cube), a view matrix (camera looking from a position), and combine: `MVP = Projection × View × Model`. Apply this to your cube vertices. Animate the rotation each frame.

**Concepts:** MVP transform chain, world space vs. camera space.

### Exercise 4.3 — Filled Cube Faces
Replace wireframe edges with filled triangle pairs (each quad face = 2 triangles). You'll immediately see a problem — faces render in wrong order. Don't fix it yet. Just observe and understand why it happens.

**Concepts:** painter's algorithm problem, why depth ordering matters.

---

## Phase 5: Depth & Visibility (Week 3–4)

### Exercise 5.1 — Z-Buffer
Allocate a float array the same size as your framebuffer, initialized to `+infinity`. Before writing a pixel, check if its interpolated Z value is less than the stored Z. If so, write the pixel and update the Z buffer. Clear it each frame.

**Concepts:** depth testing, interpolating Z across a triangle, why we store 1/z.

### Exercise 5.2 — Back-Face Culling
Before rasterizing a triangle, compute its face normal using the cross product of two edges. If the normal points away from the camera (dot product with view direction is positive), skip the triangle.

**Concepts:** winding order (CW vs CCW), surface normals, culling optimization.

### Exercise 5.3 — Near Plane Clipping
Triangles behind or straddling the camera's near plane will produce corrupt results. Implement clipping that discards or splits triangles against the near plane.

**Concepts:** Sutherland-Hodgman clipping, why geometry behind the camera breaks perspective divide.

---

## Phase 6: Shading (Week 4–5)

### Exercise 6.1 — Flat Shading
Compute a triangle's face normal. Compute the dot product of the normal with a hardcoded light direction. Multiply the triangle's base color by this value (clamp to [0,1]). Each triangle gets one uniform shade.

**Concepts:** Lambertian reflectance, dot product as cosine of angle.

### Exercise 6.2 — Gouraud Shading
Compute the lighting value per vertex instead of per triangle. Interpolate this value across the triangle using barycentric coordinates during rasterization.

**Concepts:** barycentric coordinates, attribute interpolation, vertex normals vs. face normals.

### Exercise 6.3 — Texture Mapping
Load a PNG/BMP as a pixel array (use `stb_image.h` — single header, no dependency). Add UV coordinates (u, v) to your vertices. Interpolate UVs across the triangle using barycentric coordinates, sample the texture at that UV position, and write the texel color to the screen.

**Concepts:** UV coordinates, texel sampling, perspective-correct interpolation.

### Exercise 6.4 — Bilinear Filtering
When sampling a texture, blend between the 4 nearest texels weighted by fractional distance. Compare visual quality at oblique angles.

**Concepts:** bilinear interpolation, texture aliasing, why filtering matters.

---

## Phase 7: Scene & Polish (Week 5–6)

### Exercise 7.1 — Load an OBJ File
Write a parser for Wavefront `.obj` files (plain text). Extract vertex positions, texture coordinates, normals, and face indices. Render your loaded mesh.

**Concepts:** file I/O, index buffers, shared vertices.

### Exercise 7.2 — Multiple Objects
Create a simple scene with at least 3 objects, each with their own model matrix. Ensure depth buffering handles their overlaps correctly.

**Concepts:** per-object transforms, scene management.

### Exercise 7.3 — Interactive Camera
Add keyboard/mouse input to move and rotate the camera. Rebuild the view matrix each frame from a position and direction. Walk around your scene.

**Concepts:** view matrix from position + orientation, look-at matrix.

---

## Milestone Checklist

| Exercise | Concept Unlocked |
|----------|-----------------|
| 1.1–1.3 | Window, framebuffer, loop |
| 2.1–2.4 | Math foundation |
| 3.1–3.3 | 2D rasterization |
| 4.1–4.3 | Full 3D pipeline |
| 5.1–5.3 | Depth & visibility |
| 6.1–6.4 | Lighting & textures |
| 7.1–7.3 | Complete scene |

---

## Guiding Principles

- **When stuck**, search for the concept name, not the solution.
- **Test incrementally** — after every exercise, something should be visible on screen.
- **Expect math bugs** — when something looks wrong, the issue is almost always a sign error or wrong matrix multiplication order. Print intermediate values.
