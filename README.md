<p align="center">
  <img alt="logo" src="images/logo-with-text.svg" width="50%">
  <p align="center">The worst mesh generator you'll ever use.</p>
</p>

[![PyPi Version](https://img.shields.io/pypi/v/dmsh.svg?style=flat-square)](https://pypi.org/project/dmsh/)
[![PyPI pyversions](https://img.shields.io/pypi/pyversions/dmsh.svg?style=flat-square)](https://pypi.org/project/dmsh/)
[![PyPi downloads](https://img.shields.io/pypi/dm/dmsh.svg?style=flat-square)](https://pypistats.org/packages/dmsh)



Inspired by [distmesh](http://persson.berkeley.edu/distmesh/), dmsh can be slow,
requires a lot of memory, and isn't terribly robust either.

On the plus side,

- it's got a user-friendly interface,
- is pure Python (and hence easily installable on any system), and
- it produces pretty high-quality meshes.

Combined with [optimesh](https://github.com/nschloe/optimesh), dmsh produces the
highest-quality 2D meshes in the west.

This is a somewhat modified fork of the original [dmsh](https://github.com/meshpro/dmsh) by [MeshPro](https://github.com/meshpro).
The original version of **dmsh** is [available from the Python Package
Index](https://pypi.org/project/dmsh/).

### Examples

#### Primitives

<img alt="circle" src="images/circle.svg" width="30%">

```python
import dmsh
import meshio
import optimesh

geo = dmsh.Circle([0.0, 0.0], 1.0)
X, cells = dmsh.generate(geo, 0.1)

# optionally optimize the mesh
X, cells = optimesh.optimize_points_cells(X, cells, "CVT (full)", 1.0e-10, 100)

# visualize the mesh
dmsh.helpers.show(X, cells, geo)

# and write it to a file
meshio.Mesh(X, {"triangle": cells}).write("circle.vtk")
```
<img alt="rectangle" src="images/rectangle.svg" width="30%">

```python
import dmsh

geo = dmsh.Rectangle(-1.0, +2.0, -1.0, +1.0)
X, cells = dmsh.generate(geo, 0.1)
```

<img alt="polygon" src="images/polygon.svg" width="30%">

```python
import dmsh

geo = dmsh.Polygon(
    [
        [0.0, 0.0],
        [1.1, 0.0],
        [1.2, 0.5],
        [0.7, 0.6],
        [2.0, 1.0],
        [1.0, 2.0],
        [0.5, 1.5],
    ]
)
X, cells = dmsh.generate(geo, 0.1)
```

#### Combinations

##### Difference

<img src="images/moon.svg" width="30%">

```python
import dmsh

geo = dmsh.Circle([-0.5, 0.0], 1.0) - dmsh.Circle([+0.5, 0.0], 1.0)
X, cells = dmsh.generate(geo, 0.1)
```

<img src="images/pacman.svg" width="30%">

```python
import dmsh

geo = dmsh.Circle([0.0, 0.0], 1.0) - dmsh.Polygon([[0.0, 0.0], [1.5, 0.4], [1.5, -0.4]])
X, cells = dmsh.generate(geo, 0.1, tol=1.0e-10)
```

The following example uses a nonconstant edge length; it depends on the distance to the
circle `c`.

<img src="images/rectangle-hole-refinement.svg" width="30%">

```python
import dmsh
import numpy as np

r = dmsh.Rectangle(-1.0, +1.0, -1.0, +1.0)
c = dmsh.Circle([0.0, 0.0], 0.3)
geo = r - c

X, cells = dmsh.generate(geo, lambda pts: np.abs(c.dist(pts)) / 5 + 0.05, tol=1.0e-10)
```

##### Union

<img src="images/union-circles.svg" width="30%">

```python
import dmsh

geo = dmsh.Circle([-0.5, 0.0], 1.0) + dmsh.Circle([+0.5, 0.0], 1.0)
X, cells = dmsh.generate(geo, 0.15)
```
<img src="images/union-rectangles.svg" width="30%">

```python
import dmsh

geo = dmsh.Rectangle(-1.0, +0.5, -1.0, +0.5) + dmsh.Rectangle(-0.5, +1.0, -0.5, +1.0)
X, cells = dmsh.generate(geo, 0.15)
```
<img src="images/union-three-circles.svg" width="30%">

```python
import dmsh
import numpy as np

angles = np.pi * np.array([3.0 / 6.0, 7.0 / 6.0, 11.0 / 6.0])
geo = dmsh.Union(
    [
        dmsh.Circle([np.cos(angles[0]), np.sin(angles[0])], 1.0),
        dmsh.Circle([np.cos(angles[1]), np.sin(angles[1])], 1.0),
        dmsh.Circle([np.cos(angles[2]), np.sin(angles[2])], 1.0),
    ]
)
X, cells = dmsh.generate(geo, 0.15)
```

#### Intersection

<img src="images/intersection-circles.svg" width="30%">
<table>
  <tr>
    <td>  </td>
    <td>  </td>
    <td>  </td>
  </tr>
</table>

```python
import dmsh

geo = dmsh.Circle([0.0, -0.5], 1.0) & dmsh.Circle([0.0, +0.5], 1.0)
X, cells = dmsh.generate(geo, 0.1, tol=1.0e-10)
```
<img src="images/intersection-three-circles.svg" width="30%">

```python
import dmsh
import numpy as np

angles = np.pi * np.array([3.0 / 6.0, 7.0 / 6.0, 11.0 / 6.0])
geo = dmsh.Intersection(
    [
        dmsh.Circle([np.cos(angles[0]), np.sin(angles[0])], 1.5),
        dmsh.Circle([np.cos(angles[1]), np.sin(angles[1])], 1.5),
        dmsh.Circle([np.cos(angles[2]), np.sin(angles[2])], 1.5),
    ]
)
X, cells = dmsh.generate(geo, 0.1, tol=1.0e-10)
```

<img src="images/intersection-circle-halfspace.svg" width="30%">

The following uses the `HalfSpace` primtive for cutting off a circle.

```python
import dmsh

geo = dmsh.HalfSpace([1.0, 1.0]) & dmsh.Circle([0.0, 0.0], 1.0)
X, cells = dmsh.generate(geo, 0.1)
```

### Rotation, translation, scaling

<img src="images/rotation.svg" width="50%">

```python
import dmsh
import numpy as np

geo = dmsh.Rotation(dmsh.Rectangle(-1.0, +2.0, -1.0, +1.0), 0.1 * np.pi)
X, cells = dmsh.generate(geo, 0.1, tol=1.0e-10)
```
<img src="images/scaling.svg" width="50%">

```python
import dmsh

geo = dmsh.Rectangle(-1.0, +2.0, -1.0, +1.0) + [1.0, 1.0]
X, cells = dmsh.generate(geo, 0.1)
```

```python
import dmsh

geo = dmsh.Rectangle(-1.0, +2.0, -1.0, +1.0) * 2.0
X, cells = dmsh.generate(geo, 0.1, tol=1.0e-5)
```

### Local refinement

<img alt="local-refinement" src="images/local-refinement.svg" width="30%">

All objects can be used to refine the mesh according to the distance to the object;
e.g. a `Path`:

```python
import dmsh

geo = dmsh.Rectangle(0.0, 1.0, 0.0, 1.0)

p1 = dmsh.Path([[0.4, 0.6], [0.6, 0.4]])


def edge_size(x):
    return 0.03 + 0.1 * p1.dist(x)


X, cells = dmsh.generate(geo, edge_size, tol=1.0e-10)
```

### Custom shapes

It is also possible to define your own geometry. Simply create a class derived from
`dmsh.Geometry` that contains a `dist` method and a method to project points onto the
boundary.

```python
import dmsh
import numpy as np


class MyDisk(dmsh.Geometry):
    def __init__(self):
        self.r = 1.0
        self.x0 = [0.0, 0.0]
        bounding_box = [-1.0, 1.0, -1.0, 1.0]
        feature_points = np.array([[], []]).T
        super().__init__(bounding_box, feature_points)

    def dist(self, x):
        assert x.shape[0] == 2
        y = (x.T - self.x0).T
        return np.sqrt(np.einsum("i...,i...->...", y, y)) - self.r

    def boundary_step(self, x):
        # project onto the circle
        y = (x.T - self.x0).T
        r = np.sqrt(np.einsum("ij,ij->j", y, y))
        return ((y / r * self.r).T + self.x0).T


geo = MyDisk()
X, cells = dmsh.generate(geo, 0.1)
```

### Debugging

| ![level-set-poly](https://github.com/vsukhor/dmsh/blob/main/images/levelset-polygon.png) | ![level-set-rect-hole](https://github.com/vsukhor/dmsh/blob/main/images/levelset-rect-hole.png) |
| :--------------------------------------------------------------------: | :---------------------------------------------------------------------------: |

dmsh is rather fragile, but sometimes the break-downs are due to an incorrectly defined
geometry. Use

```
geo.show()
```

to inspect the level set function of your domain. (It must be negative inside the
domain and positive outside. The 0-level set forms the domain boundary.)

### Installation

The original version of **dmsh** is [available from the Python Package
Index](https://pypi.org/project/dmsh/), so simply type

```
pip install dmsh
```

to install.

### Testing

To run the dmsh unit tests, check out this repository and type

```
tox
```

### License

This software is published under the [MIT
license](https://en.wikipedia.org/wiki/MIT_License).
