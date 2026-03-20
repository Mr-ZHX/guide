# Computer Graphics Algorithms — English Lab Manual

This English lab manual documents the “Computer_Graphics” project under:
`Computer_Graphics/`

The project demonstrates classic rasterization algorithms:

- Line drawing: `DDA` (Digital Differential Analyzer) and `Bresenham`
- Circle drawing: `Midpoint Circle Algorithm`
- Ellipse drawing: `Midpoint Ellipse Algorithm`

The implementations are written in Python and visualized with `matplotlib`. A PyQt5 GUI is provided to let you input parameters and animate the drawing process.

---

## 1. Lab Objectives

After completing and running this project, you should be able to:

1. Understand how rasterization works (drawing geometric primitives as discrete pixels).
2. Implement and compare two line rasterization algorithms:
   - `DDA` (uses incremental floating-point steps + rounding)
   - `Bresenham` (uses integer arithmetic + a decision variable)
3. Implement the midpoint algorithms for:
   - circles using symmetry (8-way)
   - ellipses using two-region decision logic (4-way symmetry)
4. Use a GUI to:
   - input endpoints/center/radii
   - animate the algorithm step-by-step with a configurable `time_gap`
   - optionally fix the axis range with `fixed_axis`

---

## 2. Project Components

### 2.1 Directory Overview

Main source files:

- `main.py`  
  GUI entry point; reads values from the interface and calls drawing functions.
- `mainGUI.py`  
  PyQt5 UI definition (auto-generated). Contains the widgets and default values.
- `dda.py`  
  Implements `DrawDDA(x1, y1, x2, y2, time_gap, fixed_axis)`.
- `Bresenham.py`  
  Implements `DrawBresenham(x1, y1, x2, y2, time_gap, fixed_axis)`.
- `midpoint_circle.py`  
  Implements `Drawcircle(x_off, y_off, r, time_gap, fixed_axis)`.
- `midpoint_ellipse.py`  
  Implements `Drawellipse(xc, yc, rx, ry, time_gap, fixed_axis)`.

---

## 3. Development Environment

The project depends on:

- Python (tested in typical environments supporting PyQt5)
- `PyQt5` for GUI
- `matplotlib` for plotting

Typical installation:

```bash
pip install PyQt5 matplotlib
```

---

## 4. How to Run the Program

1. Open `Computer_Graphics/` in your terminal.
2. Run the GUI:

```bash
python main.py
```

3. In the GUI:
   - Choose the algorithm section (DDA / Bresenham line / Midpoint circle / Midpoint ellipse)
   - Input coordinates and parameters
   - Click “绘制” (“Draw”) for the corresponding algorithm

The drawing is shown as an animated scatter plot using `matplotlib`.

---

## 5. GUI Parameter Meaning

From `main.py`, the GUI supplies the following inputs to the algorithms:

### 5.1 DDA Line Section (`drawdda`)

Parameters read from the UI:

- `DDA_start_x` (`x0`)
- `DDA_start_y` (`y0`)
- `DDA_end_x` (`x1`)
- `DDA_end_y` (`y1`)
- `time_gap` (animation pause, seconds)
- `fixed_axis` (checkbox)

It calls:
`DrawDDA(x0, y0, x1, y1, time_gap, fixed_axis)`

### 5.2 Bresenham Line Section (`drawbresenham`)

Parameters:

- `B_start_x` (`x0`)
- `B_start_y` (`y0`)
- `B_end_x` (`x1`)
- `B_end_y` (`y1`)
- `time_gap`
- `fixed_axis`

It calls:
`DrawBresenham(x0, y0, x1, y1, time_gap, fixed_axis)`

### 5.3 Midpoint Circle Section (`drawcircle`)

Parameters:

- `circle_x` (`x_off`, circle center x)
- `circle_y` (`y_off`, circle center y)
- `circle_r` (`r`, radius)
- `time_gap`
- `fixed_axis`

It calls:
`Drawcircle(x_off, y_off, r, time_gap, fixed_axis)`

### 5.4 Midpoint Ellipse Section (`drawellipse`)

Parameters:

- `ellipse_x` (`xc`, ellipse center x)
- `ellipse_y` (`yc`, ellipse center y)
- `ellipse_long` (`rx`, semi-major axis)
- `ellipse_short` (`ry`, semi-minor axis)
- `time_gap`
- `fixed_axis`

It calls:
`Drawellipse(xc, yc, long, short, time_gap, fixed_axis)`

---

## 6. Rasterization and Discrete Pixel Model

These algorithms convert ideal geometric objects (continuous lines/circles/ellipses) into a sequence of pixel coordinates (integer grid points).

Visualization details common to all algorithms:

- The plot uses `plt.scatter(x, y, color='k', s=1)` to draw each computed pixel.
- The program draws the coordinate axes:
  - a vertical line at `x = 0`
  - a horizontal line at `y = 0`
- If `fixed_axis` is enabled, the axes range is forced to:
  `[-100, 100]` for both x and y.
- After drawing each step, it calls:
  - `plt.pause(time_gap)`
  - and `plt.draw()` to refresh the figure.

---

## 7. Algorithm Descriptions and Implementation Notes

## 7.1 DDA Line Drawing (`dda.py`)

### Core Idea

The DDA algorithm incrementally steps along the major axis (the axis with larger absolute delta), using a constant increment for x and y.

### Decision of Step Count

Given endpoints `(x1, y1)` and `(x2, y2)`:

- `dx = x2 - x1`
- `dy = y2 - y1`
- The number of steps is:
  - `steps = abs(dx)` if `abs(dx) > abs(dy)`
  - otherwise `steps = abs(dy)`

### Increment and Rounding

The increments are:

- `xInc = dx / steps`
- `yInc = dy / steps`

At each step:

- `x += xInc`
- `y += yInc`
- The plotted pixel is the rounded version:
  - `ROUND(a) = int(a + 0.5)`

### Complexity

Time complexity is `O(steps)`, where `steps` is proportional to the larger coordinate difference.

---

## 7.2 Bresenham Line Drawing (`Bresenham.py`)

### Core Idea

Bresenham draws a line by using:

- integer arithmetic
- a decision variable `d` that determines whether the next pixel should move in the y direction

This avoids floating-point operations and is efficient for rasterization.

### Octant Handling (Steep Lines)

The implementation checks whether the absolute slope is greater than 1:

- If `dy > dx`, it swaps x/y (symmetric transformation) so that it can assume a “shallow” slope.
- It also updates step directions `sx` and `sy` based on the sign of `(x2-x1)` and `(y2-y1)`.

### Decision Variable Update

Initialization:

- `d = (2 * dy) - dx`

Then for each x-step (loop runs `dx` times):

- plot current pixel (after optional x/y swap)
- update `x += sx`
- if `d >= 0`:
  - `y += sy`
  - `d = d + 2*dy - 2*dx`
- else:
  - `d = d + 2*dy`

Finally, it plots the end point `(x2, y2)` and animates each step.

### Complexity

Time complexity is `O(dx)` after transforming the octant.

---

## 7.3 Midpoint Circle Algorithm (`midpoint_circle.py`)

### Core Idea

The midpoint circle algorithm exploits circle symmetry:

- For each computed point `(x, y)` in the first octant, it plots corresponding points in all 8 octants.

### Parameters

- Center: `(x_off, y_off)`
- Radius: `r`

### Decision Variable

The algorithm initializes:

- `x = 0`
- `y = r`
- `d = 1 - r`

Loop condition:

- `while x < y`

Update rules:

If `d < 0` (midpoint is inside):
- `d += 2*x + 3`

Else (midpoint is outside or on the circle):
- `y -= 1`
- `d += 2*(x - y) + 5`

Then:
- `x += 1`
- plot symmetric points
- animate with `plt.pause(time_gap)`

### Symmetry Plotting

Each iteration plots points using ±x and ±y combinations relative to the center.

---

## 7.4 Midpoint Ellipse Algorithm (`midpoint_ellipse.py`)

### Core Idea

The midpoint ellipse algorithm also uses symmetry, but the ellipse needs two region handling because the slope changes around the ellipse.

Implementation uses:

- two phases:
  - **Region 1**: from the top until the slope becomes steep enough
  - **Region 2**: from that point down to the bottom
- a decision parameter `p` updated in each region

### Parameters

- Center: `(xc, yc)`
- Semi-major axis: `rx` (called `long` in the GUI)
- Semi-minor axis: `ry` (called `short` in the GUI)

### Region 1 Initialization

Start:

- `x = 0`
- `y = ry`

Decision parameter:

- `p = ry^2 + rx^2 * (-ry + 0.25)`

Loop while:

- `ry^2 * (x + 1) < rx^2 * (y - 0.5)`

Update:

- If `p < 0`:
  - move in x direction (x increases)
  - update p using `p += ry^2 * (2*x + 3)`
- Else:
  - move in x and y directions:
  - `y -= 1`
  - update p:
    - `p += ry^2 * (2*x + 3) + rx^2 * (-2*y + 2)`

### Region 2 Initialization

After Region 1, it recomputes:

- `p = (ry*(x+0.5))^2 + (rx*(y-1))^2 - (rx*ry)^2`

Then loop:

- `while y > 0`

Update:

- If `p > 0`:
  - move in y direction:
  - update p with `p += rx^2 * (-2*y + 3)`
- Else:
  - move diagonally:
  - `x += 1`
  - update p:
    - `p += ry^2 * (2*x + 2) + rx^2 * (-2*y + 3)`
- Then `y -= 1` and plot points.

### Symmetry Plotting

Each midpoint `(x, y)` produces 4-way symmetry:

- `(xc + x, yc + y)`
- `(xc - x, yc + y)`
- `(xc + x, yc - y)`
- `(xc - x, yc - y)`

The program animates the plotting.

---

## 8. Experimental Results and Expected Observations

When you test the algorithms with different inputs, you should observe:

1. **DDA and Bresenham line outputs**
   - For shallow slopes, points should follow the line closely.
   - For steep slopes, Bresenham must correctly handle octant transformation (x/y swap).
   - Both algorithms should include the start and end points.

2. **Midpoint circle**
   - The output should be symmetric across both axes.
   - Larger radii create more points and a smoother discrete circle.

3. **Midpoint ellipse**
   - The ellipse shape should match the chosen `rx` and `ry`.
   - Symmetry across both axes should hold.
   - When `rx` and `ry` are close, the ellipse becomes closer to a circle.

---

## 9. Complexity and Practical Notes

1. **DDA**
   - Uses floating-point increments.
   - Rounding affects pixel placement.

2. **Bresenham**
   - Uses integer decision variable updates.
   - Typically faster and more exact for raster lines.

3. **Midpoint circle/ellipse**
   - Both rely on incremental decision variables and symmetry.
   - Complexity is proportional to the number of plotted points.

4. **Animation overhead**
   - The `time_gap` affects runtime and responsiveness.
   - For faster runs, use a small value or disable animation by setting a very small gap (if you modify code).

---

## 10. Conclusion

This project provides a working demonstration of classic computer graphics rasterization algorithms (DDA, Bresenham, midpoint circle, midpoint ellipse) and visualizes the point-by-point drawing process using a PyQt5 GUI and `matplotlib`.

You can use it to analyze how decision variables and symmetry reduce computation, and to compare the visual and performance characteristics of line/circle/ellipse rasterization methods.

---

End of document.

