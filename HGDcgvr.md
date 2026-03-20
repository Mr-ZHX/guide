# HIT-CGVR (Graphics & VR) — English Lab Manual (labs 1-4)

This document is an English write-up for the four labs inside:
`HIT-CGVR/lab1`, `HIT-CGVR/lab2`, `HIT-CGVR/lab3`, `HIT-CGVR/lab4`.

All labs are implemented in C++ using OpenGL 3.3 core profile with GLFW and GLAD, and rely on the bundled `include/` and `lib/` directories in each lab. Each lab also contains its own `res/` folder with shaders (and textures where needed).

---

## Common build & run notes

### Build (per lab)

Each lab has its own `CMakeLists.txt`. A typical workflow:

1. Go into a lab directory, e.g. `HIT-CGVR/lab1/`.
2. Create a build directory and build with CMake.

Example (lab1):
```bash
mkdir build
cd build
cmake ..
cmake --build . --config Release
```

### Run working directory requirement

Shaders and textures are loaded with relative paths such as:
- `res/expr1.vs`
- `res/shader/phong.vert`
- `res/diamond.jpg`
- `res/shader/circle.frag`

Therefore, run the executable with the current working directory set to the corresponding lab folder (e.g., `HIT-CGVR/lab2/`). Otherwise, the program may fail to read shader files.

### Controls summary

- `ESC`: close the window (all labs)
- Lab 2 & Lab 3: `W/A/S/D` move the camera; mouse controls view direction; mouse wheel controls zoom.
- Lab 1 & Lab 4: no camera movement (Lab 4 uses off-screen rendering).

---

## Lab 1 — Basic shader pipeline (triangle)

### Objective

Render a simple triangle using the programmable OpenGL pipeline:
- Create VAO/VBO
- Use a vertex shader and a fragment shader

### Implementation overview

Entry file:
- `lab1/src/main.cpp`

Shader wrapper:
- `lab1/src/shader.h`, `lab1/src/shader.cpp`

Vertex data:
- Three vertices in NDC space (x, y, z):
  - (-0.5, -0.5, 0.0)
  - ( 0.5, -0.5, 0.0)
  - ( 0.0,  0.5, 0.0)

Render loop:
- Clear background color
- `shader_triangle.use()`
- `glDrawArrays(GL_TRIANGLES, 0, 3)`

### Shader explanation

`lab1/res/expr1.vs`
- Takes `aPos` and sets:
  - `gl_Position = vec4(aPos, 1.0)`

`lab1/res/expr1.fs`
- Outputs a constant color:
  - `FragColor = vec4(0.1f, 0.5f, 0.8f, 1.0f)`

### Verification

1. A single solid-colored triangle should appear.
2. Changing window size should not break rendering.
3. Press `ESC` to close.

---

## Lab 2 — Textured diamond + camera (interactive)

### Objective

Render a “diamond” shape with:
- Vertex/index buffers (EBO)
- Texture loading (and texture coordinates)
- Camera movement controlled by keyboard + mouse + scroll

### Entry file & key code

- `lab2/src/main.cpp`
- `lab2/src/camera.h`
- `lab2/src/shader.h`, `lab2/src/shader.cpp`
- Resources:
  - `lab2/res/expr2.vs`
  - `lab2/res/expr2.fs`
  - `lab2/res/diamond.jpg`

### Geometry setup

The code defines:
- `diamondVertices[]`: 4D per vertex: position (3 floats) + texture coords (2 floats)
- `diamondIndices[]`: 8 triangles (drawn with `glDrawElements`)

OpenGL state:
- Depth test enabled: `glEnable(GL_DEPTH_TEST)`

### Texture setup

Texture is loaded with `stbi_load` and uploaded via:
- `glTexImage2D(...)`
- Mipmap generation: `glGenerateMipmap(GL_TEXTURE_2D)`
- Wrap: `GL_REPEAT`
- Filtering: `GL_LINEAR_MIPMAP_LINEAR` and `GL_LINEAR`

### Camera & transformation

Uniforms sent to the shader each frame:
- `model`: identity (`glm::mat4(1.0f)`)
- `view`: `camera.GetViewMatrix()`
- `projection`: perspective with:
  - `fov = camera.Zoom`
  - aspect = `SCR_WIDTH / SCR_HEIGHT`

Camera controls:
- `W/S/A/D`: `camera.ProcessKeyboard(...)`
- Mouse movement: `camera.ProcessMouseMovement(...)`
- Scroll wheel: `camera.ProcessMouseScroll(...)`

### Shader notes

Vertex shader (`lab2/res/expr2.vs`) forwards:
- positions to clip space
- texture coords to the next stage

Fragment shader (`lab2/res/expr2.fs`) outputs a constant color and does not sample the texture.

Even though `main.cpp` binds `diamond.jpg` and sets `texture1` uniform, the current fragment shader does not use it.

### Verification

1. The “diamond” (composed of triangles) should be visible with depth enabled.
2. Moving camera with `W/A/S/D` and looking with mouse should change the view direction.
3. Zoom should affect projection (narrow/wider field of view).
4. `ESC` closes the window.

---

## Lab 3 — Textured cube with Phong lighting (interactive)

### Objective

Render a textured cube with Phong illumination:
- Vertex normals transformed by the model matrix
- Ambient + diffuse + specular lighting
- Interactive camera

### Entry file & key code

- `lab3/src/main.cpp`
- `lab3/src/camera.h`
- `lab3/res/shader/phong.vert`
- `lab3/res/shader/phong.frag`
- Texture:
  - `lab3/res/yellow.jpg`

### Scene setup (from code)

Window title: `Light`

Geometry:
- A cube defined by 36 vertices (6 faces × 2 triangles × 3 vertices)
- Each vertex layout: position (3) + texcoords (2) + normal (3) = 8 floats

Render state:
- Depth test enabled

Transforms:
- `model`: translate the cube:
  - `glm::translate(glm::mat4(1.0f), glm::vec3(0.5f, 0.0f, -1.0f))`
- `view` from camera
- `projection` from perspective with `camera.Zoom`

Lighting uniforms:
- `lightColor = vec3(1.0, 1.0, 1.0)`
- `lightPos = vec3(-0.1, 1.5, 0.3)`
- `viewPos = camera.Position`

### Shader explanation

Vertex shader (`phong.vert`)
- Passes:
  - `TexCoords = aTexCoords`
  - `Normal = mat3(transpose(inverse(model))) * aNormal`
  - `FragPos = vec3(model * vec4(aPos, 1.0))`
- Sets:
  - `gl_Position = projection * view * model * vec4(aPos, 1.0)`

Fragment shader (`phong.frag`)
- Samples base color from texture:
  - `objColor = texture(texture1, TexCoords).rgb`
- Phong terms:
  - Ambient:
    - `ambient = 0.1 * lightColor`
  - Diffuse:
    - `diff = max(dot(normalized(Normal), normalized(lightPos - FragPos)), 0.0)`
    - `diffuse = diff * lightColor`
  - Specular:
    - `reflectDir = reflect(-lightDir, norm)`
    - `spec = pow(max(dot(viewDir, reflectDir), 0.0), 256)`
    - `specular = 0.5 * spec * lightColor`
- Final color:
  - `(ambient + diffuse + specular) * objColor`

### Verification

1. The cube should show visible lighting changes as you move the camera.
2. The cube surface should be textured with `yellow.jpg`.
3. The specular highlight should move with the camera position.
4. `ESC` closes the window.

---

## Lab 4 — Off-screen rendering + circle in fragment shader

### Objective

Demonstrate off-screen rendering using:
- Framebuffer Object (FBO)
- A texture attached to the FBO color attachment
- A full-screen quad to display the rendered texture
- A fragment shader that produces a circular mask using distance-from-center

### Entry file & key code

- `lab4/src/main.cpp`
- Shaders:
  - `lab4/res/shader/circle.vert`, `lab4/res/shader/circle.frag`
  - `lab4/res/shader/quad.vert`, `lab4/res/shader/quad.frag`

### Off-screen framebuffer pipeline

The program creates:

1. `framebuffer` (FBO)
2. `texColorBuffer` texture attached as:
   - `glFramebufferTexture2D(..., GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, texColorBuffer, 0)`
3. `rbo` renderbuffer attached as:
   - `GL_DEPTH_STENCIL_ATTACHMENT` with `GL_DEPTH24_STENCIL8`

Per frame:

1. Render the circle into the FBO:
   - Bind FBO: `glBindFramebuffer(GL_FRAMEBUFFER, framebuffer)`
   - Enable depth test
   - Use `circleShader`
   - Update uniform `w_div_h` based on current window size
   - Draw a screen-space quad with `GL_TRIANGLE_STRIP`
2. Render the FBO texture to the default framebuffer:
   - Bind default framebuffer: `glBindFramebuffer(GL_FRAMEBUFFER, 0)`
   - Disable depth test
   - Use `quadShader`
   - Bind `texColorBuffer` and draw the quad again

### Circle shader explanation

Vertex shader (`circle.vert`) outputs:
- `gl_Position = vec4(position, 1.0)`
- `TexCoords = texCoords`

Fragment shader (`circle.frag`)
- Converts texture coordinates to a centered coordinate system:
  - `uv = TexCoords * 2.0 - 1.0`
- Aspect correction:
  - `uv.x *= w_div_h`
- Distance to center:
  - `distance = sqrt(dot(uv, uv))`
- Creates a binary mask using `step(distance)`:
  - returns 1 inside the border (distance <= 1)
  - returns 0 outside
- Outputs `vec4(vec3(mask), 1.0)`

### Quad shader explanation

`quad.vert` passes through position and texture coordinates.

`quad.frag` samples the screen texture and outputs it to `FragColor`.

### Verification

1. The first pass should generate a circle mask texture (off-screen).
2. The second pass should display that texture on the window.
3. Resizing the window should preserve the circle shape via `w_div_h`.
4. `ESC` closes the window.

---

## Suggested submission content (English)

For a complete submission, include:

- For each lab:
  - a short objective statement
  - key implementation details (buffers, shaders, uniforms)
  - screenshot(s) showing the output
  - explanation of controls (if applicable)
- For Lab 3:
  - include a description of the Phong lighting computation
- For Lab 4:
  - include an explanation of FBO + texture-based full-screen quad rendering

---

