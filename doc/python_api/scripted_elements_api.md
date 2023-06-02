# Scripted elements API

This reference describes functions and data structures necessary for the implementation of "scripted elements".

```{seealso}
If you don't know about the concept yet, take a look at the [Scripted elements introduction](../howtos/scripted_elements/scripted_elements_introduction.md).
```

## The `dialog` function

💡 **Notes:**

* The main purpose of this function is to use display a script dialog and allow the user to enter element parameters.
* All tokens of other elements can be accessed within this function.

### Signature

```python
def dialog(context, params):
```


#### Parameter `context`

The context contains the following members:

| Name                       | Role                                               | Remarks                                                        |
| -------------------------- | -------------------------------------------------- | -------------------------------------------------------------- |
| <pre>.data</pre>           | UDE data storage access                            | see "calculate" Function                                       |
| <pre>.stages</pre>         | Current stage                                      | List containing stage index of current stage                   |
| <pre>.total_stages</pre>   |	Number of stages                                  | Valid stage indices are 0 to total_stages - 1                  |
| <pre>.calc</pre>           | Function to calculate preview                      | See below                                                      |
| <pre>.result</pre>         | Directly set preview                               | see "calculate" Function                                       |
| <pre>.is_new_element</pre> | Flag if element creation                           | True on (edit) creation, false on recalc                       |
| <pre>.name</pre>           | Name of created element                            | Read/write attribute<br>Ignored on recalc and script execution |
| <pre>.error</pre>          | Last error text from preview calculation           | Empty if no error occurred                                     |
| <pre>.edit_element</pre>   | Reference to edited element                        | Only available during element editing                          |
| <pre>.recalc_element</pre> | Reference to element used in project recalculation |                                                                |

#### Parameter `params`

The params contain a map of parameter values. It is only filled on element edit and contains the current parameters of the element.

#### Return value

The function must return a map containing the new element parameters.

If no map is returned the edit/creation is regarded as aborted by the user.

### Calculating a preview

To calculate a preview, the `context.calc()` function can be invoked. It takes the following parameters:

| Name              | Role                                                                               |
| ----------------- | ---------------------------------------------------------------------------------- |
| <pre>params</pre> | A map of parameters to be used for preview calculation                             |
| <pre>stages</pre> | Optional: A list of stage indices to calculate a preview in                        |
| <pre>dialog</pre> | Optional: A script dialog to message when preview calculation was successful.<br>The dialog event handler will receive the event string 'calculated' |

A call to this function will return immediately. The calculation is invoked asynchronously in a separate python instance.

## The `calculation` function

💡 **Notes:**
* It is not possible to call script commands or read tokens from within this function. (Do not call `gom.app.project....`)
* The function should loop over all stages to be calculated and set a computation result  for each stage.

### Signature

```python
def calculation(context, params):
```

#### Parameter `context`

The context contains the following members:

| Name                                  | Role                                               | Remarks                                                        |
| ------------------------------------- | -------------------------------------------------- | -------------------------------------------------------------- |
| <pre>.data[stage]</pre>               | UDE data storage access                            | see below                                                      |
| <pre>.stages</pre>                    | Current indices                                    | List containing stage indices to calculate                     |
| <pre>.total_stages</pre>              | Number of stages                                   | Valid stage indices are 0 to total_stages - 1                  |
| <pre>.result[stage]</pre>             | Directly set preview                               | see below                                                      |
| <pre>.is_new_element</pre>            | Flag if element creation                           | True on (edit) creation, false on recalc                       |
| <pre>.name</pre>                      | Name of created element                            | Read/write attribute<br>Ignored on recalc and script execution |
| <pre>.error[stage]</pre>              | Used to assign an error text                       | Will set the element to not computed in the given stage        |
| <pre>.edit_element</pre>              | Reference to edited element                        | Only available during element editing                          |
| <pre>.recalc_element</pre>            | Reference to element being computed                | Only available during non-interactive calculation              |
| <pre>.progress_stages_computing</pre> | Number of stages which have started to compute     | Used to display progress information                           |
| <pre>.progress_stages_total</pre>     | Number of stages which have to be computed         | Used to display progress information                           |

##### Attribute `context.data[]`

The context.data is a list allowing to read or write additional data. The list is indexed with a stage index. The additional data is stored within the project, so the gom application must be able to serialize the provided data. 

```Python
context.data[0] = value
value = context.data[0]
```

##### Attribute `context.result[]`

This is a write only list used to set computation results. The context.result[] should be set for each state index listed in context.stages.

The format to write must match the type of the script element. For available result types, see [Scripted actuals - Return values](#scripted-actuals---return-values) and [Scripted checks - Return values](#scripted-checks---return-values).

#### Return value

On success the function must return True, otherwise False.

% `context`: ...


## Scripted actuals - Return values

| Dialog Choice  | Comment                  | Result format                                              | Result description |
| -------------- | ------------------------ | ---------------------------------------------------------- | ------------------ |
| Point          | Plain 3D point           | <pre>result = (x,y,z)</pre><br><pre>result = gom.Vec3D</pre>                 | Point coordinate   |
| Distance	     | Two point distance       | <pre>result = { 'point1': (x,y,z), 'point2': (x,y,z) }</pre><br><pre>result = { 'point1': gom.Vec3D, 'point2': gom.Vec3D }</pre> | Start and end point of distance |
| Value Element	 | Plain value (only real values supported) | <pre>result = x</pre>                               | any double value   |
| Circle	     | 2D Circle with direction | <pre>result = { 'center' : gom.Vec3D, 'direction' : gom.Vec3D, 'radius' : double }</pre> | A center point, direction vector and radius (double) |
| Curve	         | 3D polyline	           | <pre>result = [ { 'points': [gom.Vec3D, gom.Vec3D, ...] } ]</pre>   | A curve a made up by an array of subcurves. Each subcurve is a polyline.<br>A closed curve will be created, if first point = last point. |
| Surface Curve	 | 3D polyline with normals | <pre>result = [ { 'points': [ gom.Vec3D, gom.Vec3D, ... ], 'normals': [(x,y,z)] } ]</pre> | Like a curve with additional normal data. |
| Section	     | 3D polyline with normals | <pre>result = {'curves': [{'points': [(x, y, z), ...], 'normals': [(x, y, z), ...]}, ...], 'plane' : {'normal' : (x, y, z), 'distance' : float}, 'cylinder': ..., 'cone' : ...}</pre> | Parameters 'plane', 'cylinder' and 'cone' are optional. They denote the creation geometry. You can only use one of them. Argument is a corresponding trait. |``
| Point Cloud	 | Set of 3D points	       | <pre>result = { 'points' :  [ gom.Vec3D, gom.Vec3D, ... ] , 'normals' : [ gom.Vec3D, gom.Vec3D, ... ] }</pre> | A set of points. The 'normals 'attribute is optional. |
| Surface	     | Mesh                     | <pre>result = { 'vertices': [ (x,y,z) ], 'triangles':  [ (v0,v1,v2) ] }</pre> | Defines a triangulation. The vertices attribute points defines all points. The triangle attribute defines triangles between these points using indices into the vertex list. |
| Cone	         | Cone                     | <pre>result = Reference</pre><br><pre>result = { 'normal' : gom.Vec3d, 'distance': float }</pre><br><pre>result = { 'target': Reference, 'offset': float }</pre><br>⚠️ **The creation of planes currently does not work.**<br>**Workaround:** set the result to <pre>{'default' : {'normal' : gom.Vec3d, 'distance': float} }</pre> | Accepts any Plane Trait |
| Cylinder	     | Cylinder	               | <pre>result = Reference</pre><br><pre>result = { 'point': gom.Vec3d, 'radius': float, 'direction': gom.Vec3d, 'inner' : bool } # This does not work!</pre><br>⚠️ **Workaround:** set the result to <pre>{'default' : {'point': gom.Vec3d, 'radius': float, 'direction': gom.Vec3d, 'inner' : bool} }</pre> | Accepts any Cylinder Trait |
| Plane	         | Plane                    | <pre>result = Reference</pre><br><pre>result = { 'point1': gom.Vec3d, 'radius1': float, 'point2': gom.Vec3d, 'radius2': float } # This does not work!</pre><br>⚠️ **Workaround:** set the result to <pre>{'default' : {'point1': gom.Vec3d, 'radius1': float, 'point2': gom.Vec3d, 'radius2': float} }</pre> | Accepts any Cone Trait |
| Volume defects | Volume defects element   | <pre>result = {<br>   'vertices': [ np.array((x,y,z), (x,y,z), ... ), np.array((x,y,z), (x,y,z), ...), ... ],<br>   'triangles':  [ np.array((v0,v1,v2), (v0,v1,v2), ... ), np.array((v0,v1,v2), (v0,v1,v2), ...), ... ],<br>   'outer_hull' : gom.Reference     # optional OR<br>   'outer_hull_vertices': np.array((x,y,z),...), 'outer_hull_triangles': np.array((v0,v1,v2),...) }<br>}</pre> | A list of meshes defined by vertices and triangles.<p>The vertices attribute is a [python array] – one entry for each defect –  of numpy arrays (np.array) of Vec3d.<p>The triangle attribute defines triangles between the points of each mesh using indices to the vertex lists.<p>The 'outer_hull' parameter can optionally be set to a reference of a mesh element of the project. This mesh will be copied and used as an outer hull for the defect element. **Alternatively**, 'outer_hull_vertices' and 'outer_hull_triangles' can be given as explicit outer hull mesh definition.<p>For an example, see: [\[GOM Connect\] How-to: Generate volume defect elements from GOM Scripting](https://connect.gom.com/x/B8fVAg) |
| 2D Volume Defects	| 2D volume defects element of curves<p>needed for the P201 package | <pre>result = { 'curves': [ [gom.Vec3d, gom.Vec3d, ...],<br>                       [gom.Vec3d, gom.Vec3d, ...],<br>                        ... ] }</pre> | Requires a list/array of lists/arrays of Vec3ds.<p>A list of points represents the polygon (curve) of one 2d volume defect. The list of lists of points represents all 2d volume defects that shall be included in the element. |
| Volume	        | New volume data       | <pre>result = { 'voxel_data' : np.array (), 'transformation' : (gom.Mat4x4 \| gom.Vec3d) }</pre> | Accepts a numpy array with voxel data and a transformation. <p>The numpy array's shape denotes the resulting volume shape. The 'dtype' can be one of (UINT16, BYTE, INT16, INT16, INT32, UINT32, FLOAT, DOUBLE).<p>The transformation can be a gom.Mat4x4 (affine transformation) or a gom.Vec3d (scaling along x/y/z axis only) |
| Volume material map | Attach material labels to volume element | <pre>result = {<br>    'material_labels_draft' : np.array (), <br>    'material_grey_values_draft' : [background, material 1, ...],<br>    'volume_reference_draft' : Reference<br>}</pre> | Creates a new volume element copy with attached material labels.<p>First parameter is a numpy array of type UINT8 of the size of the volume. The values are the material index per voxel. Background has Index 0.<p>The second parameter is a list of floating point grey values that are the representative grey values of the background and the materials.<p>The third parameter is a reference to the volume element, to which material labels should be attached. |
| Volume Section    | Volume section        | <pre>result = { 'pixel_data' : np.array (), 'transformation' : gom.Mat4x4 }</pre> | Accepts a numpy array with pixel data and a transformation.<p>The numpy array's shape denotes the resulting volume section shape. The 'dtype' must be FLOAT.<p>The transformation is a gom.Mat4x4 (affine transformation) |
| Volume Region	    | Volume Region         | <pre>result = {'volume_element': Reference,<br>          'offset': gom.Vec3d,<br>          'data': np.array ()}</pre> | Accepts a numpy array of the region data. The 'dtype' must be UINT_8. This array can be smaller than the volume grid.<p>The offset parameter defines the location of the first voxel in the numpy array of the volume region.<p>This scripted element requires specifying a reference to a volume element. This can be a volume or linked volume element. |


% ### Element type: Point

% ```{py:function} context.result[s]
% 
% Test if the referenced element is suitable for inspection with a scalar check.
% 
% :API version: 1
% :param element: Element to be checked
% :type  element: Reference
% :return: True' if the given element s suitable for inspection with a scalar check.
% :rtype: bool
% ```

## Scripted checks - Return values

Scripted Checks extend the concept of scripted actual elements to be able to calculate custom inspections based on python scripts.

### Supported Element Types

The following actual element types are currently supported:

| Dialog Choice | Comment | Result format | Result description |
| ------------- | ------- | ------------- | ------------------ |
| Scalar        | Scalar check:<br>A single scalar value is assigned to an element | `result = {"nominal" : double, "actual" : double, "reference" : gom.Reference }` | A nominal and actual double value are set, and a reference to element which was checked. |
| Scalar Surface | Surface check:<br>A scalar deviation value is assigned to each point of a mesh. | `result = { "deviation_values" : np.array(dtype=np.float32), "reference" : gom.Reference }` | A list of deviation values for each point of a mesh. The mesh is also set as "reference" parameter.<p>The number of points of the mesh and the "deviation_values" array must match. |
| Scalar Curve | Curve check:<br>A scalar deviation value is assigned to each point on a curve. | `result = { "actual_values" : double, 'nominal_values': double, "reference" : gom.Reference}` | A list of nominal and actual values for each point of a curve. The deviation values are calculated automatically as a difference between both.<p>The curve is also set as "reference" parameter.<p>The number of points of the curve and the value arrays must match. |

