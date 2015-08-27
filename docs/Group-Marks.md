> [Wiki](Home) ▸ [[Documentation]] ▸ **Group Marks**

Group marks are a special kind of mark that can contain other marks. A group mark definition is in many ways similar to the top-level [Visualization](Visualization) definition: a group can contain scales, axes, legends and marks. However, note that one can not set top-level `width`, `height`, `padding` or `data` definitions. Instead, group marks use the same `"from"` data definition and visual property sets as other marks.

Group marks can be used in a variety of designs. For example:
* To incorporate multiple visualizations within the same Vega specification.
* To create layered or stacked visualizations, in which each layer or stack contains a different subset of data.
* To create small multiples displays in which a visualization design is systematically repeated over different data subsets.

### Basic Visual Properties

Group marks support the same visual properties as `rect` marks. For instance, group marks can have `x`, `y`, `width`, and `height` values, as well as fill and stroke properties. By default, group marks have no stroke or fill, and inherit the spatial properties of their parent group or top-level visualization.

### Backing Data

Group marks are populated from data just like any other mark: the `from` property defines a source data set along with any desired data transforms. One group instance will be created for each element in the backing data set. If no `from` property is specified, the group will receive data from its parent group (if it is nested within another group) or will be backed by a single null value (if it is a top-level group).

Groups differ from other mark types in their ability to contain children marks. Marks defined within a group mark can _inherit_ data from their parent group. For inheritance to work each data element for a group must contain data elements of its own. This arrangement of nested data is typically achieved by _facetting_ the data, such that each group-level data element includes its own array of sub-elements. Facets can be constructed using the __facet__ or __window__ data transforms (see the [Data Transforms](Data-Transform) page) or can be loaded directly as hierarchical data (e.g., using the __treejson__ data type, see the [Data](Data) page for more).

### Scales, Axes and Legends

Groups can also contain their own `scales`, `axes`, and `legends` definitions. These definitions are the same as those used in a top-level visualization definition, barring a few simple differences:

* Scales, axes and legends defined at the group level reference the width and height values of the current group, _not_ the top-level visualization.
* If a scale is defined using the same name as a previously defined scale, the pre-existing scale will be shadowed (overloaded) by the new definition within the context of this group. Scale definitions cascade, so that any (non-shadowed) scales defined at a higher level are still accessible.
* Scale domain definitions can omit the (normally required) `"data"` property. If no `"data"` property is provided in the domain definition, the group-level data will be used to determine the domain. Note that this group-level data is exactly the same data that gets passed along to child marks.

##### Examples

Groups are perhaps more easily understood by example. The visualization specifications included with the online Vega editor showcase a number of use cases for groups, including

| Example                                      | Description                         |
| :------------------------------------------- | :---------------------------------- |
| [stacked_area](http://vega.github.io/vega/editor/index.html?spec=stacked_area) [jobs](http://vega.github.io/vega/editor/index.html?spec=jobs) | use a __facet__ transform to gather data into individual series (or stacks). A group mark is then used to layer each stack. In this case, each group has the same position.|
| [barley](http://vega.github.io/vega/editor/index.html?spec=barley)| uses facets and group marks, but rather than layering data, each sub-group is placed in its own plot to create a small multiples display. In addition, a group-level scale is used to conveniently place items along the y-axis in each plot.|
| [grouped_bar](http://vega.github.io/vega/editor/index.html?spec=grouped_bar) | uses the group mark to subdivide data to create a grouped bar chart display.|
| [scatter_matrix](http://vega.github.io/vega/editor/index.html?spec=scatter_matrix) | uses the __cross__ transform and group marks to create a scatter plot matrix display.|

