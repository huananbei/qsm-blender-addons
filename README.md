# QSM Blender Addons

Blender addon for importing cylindrical quantitative structure models (QSM) and leaf models (L-QSM). QSM is a description of the topology and geometry of a tree. For example the [TreeQSM](https://github.com/InverseTampere/TreeQSM) method can be used to reconstruct QSMs for terrestrial laser scanning data. These tree models can be augmented with a leaf cover, with the [QSM-FaNNI](https://github.com/InverseTampere/qsm-fanni-matlab) algorithm. The QSM-blender-addon can be used to import the resulting geometry from both procedures into [Blender](https://www.blender.org/).

In the future the repository may be extended with additional addons related to QSMs. However, at the moment the repository only covers the import addon.

## Description

The import addon can be used to import cylindrical QSMs and leaf models, using a graphical user interface added to the 3D view tool shelf. The required input data formats and options are described below.

## Importing QSMs / tree models

At this time, the addon only imports QSM geometry while ignoring most of the topology. Therefore, the input data format is a TXT-file, where each line describes a single cylinder of a QSM. The order of the values on a single row, is the following:

- branch index (1)
- starting point (3)
- axis direction (3)
- length (1)
- radius (1)
- optional parameters (N)

The number of elements used to describe the property is given in parenthesis. Branch index is used to define which cylinders form a branch. The next four parameters define the cylinder geometry. The optional parameters can be used to import color data for mesh based objects, as described later. As example the following text content describes the test QSM used in the [QSM-FaNNI](https://github.com/InverseTampere/qsm-fanni-matlab) repository, in the described format.

```
1  0.0000  0.0000 0.0000  0.0000  0.0000 1.0000 1.0000 0.3000
1  0.0000  0.0000 1.0000  0.7070  0.0000 0.7070 1.4140 0.2000
1  1.0000  0.0000 2.0000  1.0000  0.0000 0.0000 1.0000 0.1000
2  0.0000  0.0000 1.0000  0.0000  0.7070 0.7070 1.4140 0.2000
2  0.0000  1.0000 2.0000  0.0000  1.0000 0.0000 1.0000 0.1000
3  0.0000  0.0000 1.0000  0.0000 -0.7070 0.7070 1.4140 0.2000
3  0.0000 -1.0000 2.0000  0.0000 -1.0000 0.0000 1.0000 0.1000
4  0.0000  0.0000 1.0000 -0.7070  0.0000 0.7070 1.4140 0.2000
4 -1.0000  0.0000 2.0000 -1.0000  0.0000 0.0000 1.0000 0.1000
```

When running Blender, the import panel will be visible in the tool shelf of the 3D view under the title *QSM Import*. The user has the option to choose the imported object type from three options: 

1. mesh object
2. cylinder-level Bezier curves
3. branch-level lofted Bezier curves

The appearance of the UI depends on the selected import type. The resulting render with all three import types with the above example data, is shown below.

![Addon mesh UI](https://github.com/InverseTampere/qsm-blender-addons/raw/master/qsm-addon-example.png)

Examples of the UI are given in the subsection. Regardless of the import type, the common inputs for the import procedure are the following:

Name | Type | Description
---|---|---
Input file | File path | Path to a input file with cylinder parameters in the above format. The *Browse* button can be used to open a graphical file browsing view.
Stem material | Material name | Material to be applied to the cylinders of the branch with the lowest branch index. If no *Branch material* is given, *Stem material* will be applied to all cylinders. Selecting a stem material is optional.
Branch material | Material name | Material to be applied to cylinders not part of the stem branch. Selecting a branch material is optional.
Branch separation | Checkbox | Import individual branches as separate Blender objects. If unchecked the import results in a single object.

Once all the parameters have been selected, the import procedure is started using the *Import* button at the bottom of the panel. The addon will create an empty that will act as the parent of either the single resulting object, when branch separation is deactivated, or all the resulting branch object, when activated.

### Mesh import

![Addon mesh UI](https://github.com/InverseTampere/qsm-blender-addons/raw/master/qsm-addon-ui-mesh.png)

The level-of-detail for imported cylinder meshes can be selected with the *Vertex count* parameter. Vertex count is defined as the number of vertices on a single circular loop of a cylinder, *i.e.*, the number of faces of the cylinder envelope. The two element vector parameter can be used to set the minimum and maximum vertex counts. The vertex count value is selected for each cylinder during the import, by linearly interpolating between the minimum and maximum vertex counts. The interpolation is defined as:

```python
nvert = vmin + (vmax-vmin)*(r-rmin)/(rmax-rmin)
```

where `nvert` is the selected vertex count, `vmin` and `vmax` are the minimum and maximum vertex counts selected by the user, respectively, `r` is the radius of the given cylinder and `rmin` and `rmax` are the minimum and maximum radius values given in the input file.

Internally the addon relies on the function `bpy.ops.mesh.primitive_cylinder_add()`. When the function is called the parameter `end_fill_type` is set to `'NGON'`, thus resulting in closed cylinders, *i.e.*, cylinders with ngons as their bottom and top planes.

### Coloring meshes

As mentioned above, additional parameters can be put at the end of the lines of the input text file for color information. The values can be either a single decimal between zero and one, or three. In both cases, a custom color layer is attached to the vertex data of the resulting object. The name of the layer is `Color` and it can be accessed with the *Attribute* node in the Node editor, for example as follows:

![Material example](https://github.com/InverseTampere/qsm-blender-addons/raw/master/qsm-addon-vertex-coloring.png)

When only one value is present instead of three, that value is replicated in all three elements of the color value. With three values the vector is stored directly in the color layer.

Vertex coloring is currently not available for curve objects.

### Bezier import

![Addon Bezier UI](https://github.com/InverseTampere/qsm-blender-addons/raw/master/qsm-addon-ui-bezier.png)

With the two Bezier curve based import types, the level-of-detail does not need to be fixed upon import. Instead a lofting object is used to define the cross-section shape around the Bezier curve defining either a single cylinder or a complete branch. The lofting object is set with the *Bevel Object* parameter that is an object selector. The lofting object should be a curve object, *e.g.*, a Bezier circle with a unit radius. The `Resolution` of the curves spline can be used to change the level-of-detail of all the cylinders in the resulting model, in the objects *Object data* panel.

Selecting *Bezier cylinder* as the import type, creates a single Bezier spline, with two curve points at the starting and ending point of the each cylinder. The radius value of the spline will be set to the radius value of the cylinder.

Selecting *Bezier branch* as the import type, creates a single Bezier spline per each branch. Curve points are placed at the center points of each cylinder, as well as, the first point at the starting point of the first cylinder, and the last point at the ending point of the last cylinder. At the last point the curve radius is set as 10% of the radius of the last cylinder. At the other curve points the radius is set as the radius of the respective cylinder, creating smooth tapering along the curve.

## Importing leaf models

![Leaf import UI](https://github.com/InverseTampere/qsm-blender-addons/raw/master/qsm-addon-ui-leaves.png)

Leaf models are imported from a text file following the Wavefront OBJ-format, that defines vertices and indices of vertices forming faces. Each leaf is expected to have the same basis geometry. The input parameters are the following:

Name | Type | Description
---|---|---
Input file | File path | Path to a input file with leaf geometry. The *Browse* button can be used to open a graphical file browsing view.
Material | Material name | Material applied to the leaves. Selecting a material is optional.
Vertex count | Integer | Number of vertices on a single leaf. The number is used to create a UV-map where each leaf is overlaid on the UV-plane. Currently supported values are 3 and 4.

After filling in the required parameters, the import process is initiated using the *Import leaf model* button. An example of a rendered, imported leaf model and a QSM can be seen below.

![Example render](https://github.com/InverseTampere/qsm-fanni-matlab/raw/master/src/test_result.png)
