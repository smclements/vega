## DATA TRANSFORMS

A data transform performs operations on a data set prior to visualization. Common examples include filtering and grouping (e.g., group data points with the same stock ticker for plotting as separate lines). Other examples include layout functions (e.g., stream graphs, treemaps, graph layout) that are run prior to mark encoding.

All transform definitions must include a `type` parameter, which specifies the transform to apply. Each transform then has a set of transform-specific parameters. Transformation workflows are defined as an array of transforms; each transform is then evaluated in the specified order. 

These workflows can be specified as part of either a dataset definition:

```json
  "data":[
    {
      "name": "stats",
      "source": "table",
      "transform": [
        {"type": "facet", "groupby": ["x"]}
      ]
    }
  ]
```

or added to a mark's `from` property:
```json
  "marks": [
    {
      "type": "group",
      "from": {
        "data": "fields",
        "transform": [{"type": "cross"}]
      },
      ...
    }
  ],
```

Many transforms accept data attributes, or fields (denoted below as "Field"), as parameters. Data field parameters are strings that describe either individual attributes (e.g., `"price"`) or access paths (e.g., `"price.min"`). Data fields can also access array indices: for example `"list.0"` is equivalent to `list[0]` in normal JavaScript.

### Data Manipulation Transforms

These **transforms** can be used to manipulate the data: to filter and sort elements, form groups, and merge different data sets together.

[aggregate](#-aggregate), [bin](#-bin), [cross](#-cross), [facet](#-facet), [filter](#-filter), [fold](#-fold), [formula](#-formula), [sort](#-sort), [zip](#-zip)


### aggregate

Computes aggregate summary statistics (e.g., median, min, max) over groups of data.

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| groupby             | Array&lt;String&gt; | An optional array of fields by which to group data values by.|
| summarize           | JSON                | Summary aggregates to compute for each group. This property supports two formats: a convenient short format, and a more complete long format.| 

The short format uses a single object hash that maps from field names to one or more aggregation operations: `{"foo": "mean", "bar": ["sum", "median"]}`. The aggregation operation can be either a single string or an array of strings, each a valid aggregation operation name.

The long format uses an array of aggregate specification objects. The previous short format example translates to the following long format: `[{"field": "foo", "ops": ["valid"]}, {"field": "bar", "ops": ["sum", "median"]}]`

An __aggregate specification__ supports the following properties:

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| field               | String              | The name of the field to aggregate. This name will be used to generate output field names, unless a custom name is specified (below).|
| ops                 | Array               | An array of aggregate operations. See below for supported aggregates.|
| * _as_              | Array               | An optional array of names to use for the output properties. By default, the aggregator will automatically create output field names of the form _op_name_ (e.g., `sum_bar`, `median_bar`). The _as_ array provides a set of custom names to use instead. The array should be the same length as the _ops_ array. Standard automatic name generation is used for `null` entries.|

The supported **aggregation operations** are:

| Operation       | Description  |
| :---------------| :------------|
| values          | Builds up an array of all input objects in the group.|
| count           | Count the total number of elements in the group.|
| valid           | Count values that are not `null`, `undefined` or `NaN`.|
| missing         | Count the number of `null` or `undefined` values.| 
| distinct        | Count the number distinct values.| 
| sum             | Compute the sum of values in a group.| 
| mean            | Compute the mean (average) of values in a group.|
| average         | Compute the mean (average) of values in a group. Identical to _mean_.|
| variance        | Compute the sample variance of values in a group.|
| variancep       | Compute the population variance of values in a group.|
| stdev           | Compute the sample standard deviation of values in a group.|
| stdevp          | Compute the population standard deviation of values in a group.|
| median          | Compute the median of values in a group.|
| q1              | Compute the lower quartile boundary of values in a group.|
| q3              | Compute the upper quartile boundary of values in a group.|
| modeskew        | Compute the mode skewness of values in a group.|
| min             | Compute the minimum value in a group.|
| max             | Compute the maximum value in a group.|
| argmin          | Find the input object that minimizes the value in a group.|
| argmax          | Find the input object that maximizes the value in a group.|

Many of the aggregation functions above are straightforward, but a few deserve additional discussion. 

The _'values'_ and _'count'_ functions operate directly on the input objects and return the same value regardless of the provided field name. Similar to SQL's `count(*)`, these can be specified with the special name `"*"`, as in `"summarize": {"*": "count"}`.

The _'argmin'_ and _'argmax'_ functions are a bit unusual: instead of returning the minimum or maximum value of a field, they return the original input object that contains the minimum or maximum value. This can be useful for retrieving another field associated with the minimum or maximum value (e.g., for each region, in which year did I have the maximum revenue?). If multiple entries share the minimum or maximum value, the first observed input object will be returned.


The __aggregate__ transform outputs a new array of data objects, one for each group, with the computed aggregate statistics. 

## Example

For the following input data:
```json
[{"foo": 1, "bar": 1}, {"foo": 1, "bar": 2}, {"foo": null, "bar": 3}]
```

This short format aggregate transform
```json
{"type": "aggregate", "summarize": {"foo": "valid", "bar": ["sum", "median"]}}
```
would produce the following output:
```json
[{"valid_foo": 2, "sum_bar": 6, "median_bar": 2}]
```

Similarly, this long format aggregate transform:
```json
{
  "type": "aggregate",
  "summarize": [
    {"field": "foo", "ops": ["valid"]},
    {"field": "bar", "ops": ["sum", "media"], "as": ["s", "m"]}
  ]
}
```
would produce the following output:
```json
[{"valid_foo": 2, "s": 6, "m": 2}]
```


### bin

Bins raw data values into quantitative bins (e.g., for a histogram).

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| field               | String              | The name of the field to bin values from.|
| min                 | Number              | The minimum bin value to consider.|
| max                 | Number              | The maximum bin value to consider.|
| base                | Number              | The number base to use for automatic bin determination (default is base 10).|
| maxbins             | Number              | The maximum number of allowable bins.|
| step                | Number              | An exact step size to use between bins. If provided, options such as maxbins will be ignored.|
| steps               | Array               | An array of allowable step sizes to choose from.|
| minstep             | Number              | A minimum allowable step size (particularly useful for integer values).|
| div                 | Array               | Scale factors indicating allowable subdivisions. The default value is [5, 2], which indicates that for base 10 numbers (the default base), the method may consider dividing bin sizes by 5 and/or 2. For example, for an initial step size of 10, the method can check if bin sizes of 2 (= 10/5), 5 (= 10/2), or 1 (= 10/(5*2)) might also satisfy the given constraints.|

The __bin__ transform returns the input data set, with an additional property, `bin`, that contains the binned value for the specified `field`. This property may be renamed by specifying an `output` parameter like so: `"output": {"bin": "b"}`.

## Example

```json
{"type": "bin", "field": "amount", "min": 0, "max": 10, "maxbins": 5}
```

This example will bin values in the `amount` field into one of 5 bins between 0 and 10.
Given the following input data:
```json
[
  {"amount": 3.7},
  {"amount": 6.2},
  {"amount": 5.9},
  {"amount": 8},
]
```

The bin transform produces the following output:
```json
[
  {"amount": 3.7, "bin": 2},
  {"amount": 6.2, "bin": 6},
  {"amount": 5.9, "bin": 4},
  {"amount": 8, "bin": 8},
]
```


### cross

Compute the cross-product of two data sets.

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| with                | String              | The name of the secondary data set to cross with the primary data. If unspecified, the primary data is crossed with itself.|
| diagonal            | Boolean             | If false, items along the "diagonal" of the cross-product (those elements with the same index in their respective array) will _not_ be included in the output. This parameter is true by default.|


The __cross__ transform outputs an array of data objects with two properties: one containing an item from the primary data set (named `a` by default), and an item from the secondary data set (named `b` by default). These names can be changed by setting the transform's output map. For example, the parameter `"output": {"left":"thing1", "right":"thing2"}`, causes the properties `thing1` and `thing2` to be used instead of `a` and `b`.

## Example

```json
{"type": "cross", "diagonal": false}
```
This example crosses a data set with itself, ignoring entries with the same indices (e.g., along the diagonal). If the input data is `[1, 2, 3]`, then the __cross__ transform will output:
```json
[
  {"a":1, "b":2},
  {"a":1, "b":3},
  {"a":2, "b":1},
  {"a":2, "b":3},
  {"a":3, "b":1},
  {"a":3, "b":2},
]
```


### facet

Organizes a data set into groups or "facets". The __facet__ transform is useful for creating collections of data that are then passed along to group marks to create hierarchical structure in a visualization. It can also be used (like the __aggregate__ transform) to compute descriptive statistics over subgroups of data. In this sense, it is similar to a "group by" operation in SQL.

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| groupby             | Array&lt;Field&gt;  | The fields to use as keys. Each unique set of key values corresponds to a single facet in the output.|
| summarize           | JSON                | The summary aggregates to compute for each subgroup. See the [aggregate transform](#-aggregate) for more information.|
| transform           | Array&lt;Transform&gt;| A workflow of data transformations to apply to each subgroup.| 


The __facet__ transform returns a transformed data set organized into facets. Vega uses a standardized data structure for representing hierarchical or faceted data, which consists of a hierarchy of objects with __key__ and __values__ properties.

## Example
```json
{"type": "facet", "keys": ["category"]}
```
Facets the data according to the values of the `category` attribute. Given the following input data:

```json
[
  {"category":"A", "value":0},
  {"category":"B", "value":1},
  {"category":"A", "value":2},
  {"category":"B", "value":3}
]
```

The facet transform produces a hierarchical collection of data arrays:
```json
[
  {
    "category": "A",
    "key": "A",
    "values": [{"category":"A", "value":0}, {"category":"A", "value":2}]
  },
  {
    "category": "B",
    "key": "B",
    "values": [{"category":"B", "value":1}, {"category":"B", "value":3}]
  }
]
```

Each facet (group) is bundled into a "sentinel" object that includes:
* The original key properties and values (`category` in this example).
* A string concatenating all key values (`key`). This can be useful in conjunction with ordinal scales.
* The array of grouped data objects (`values`).

When a faceted data set is the input data for a group mark, Vega will automatically lookup the `values` array and pass it down as the data source for any marks contained within each enclosing group mark instance.


### filter

Filters elements from a data set to remove unwanted items.

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| test                | [Expression|(#expression) | A string containing an expression (in JavaScript syntax) for the filter predicate. The [[expression language|Expressions]] includes the variable `datum`, corresponding to the current data object.|


The __filter__ transform returns a new data set containing only elements that match the filter _test_.

## Examples

```json
{"type": "filter", "test": "datum.x > 10"}
```
This example retains only data elements for which the field `x` is greater than 10.

```json
{"type": "filter", "test": "log(datum.y)/LN10 > 2"}
```
This example retains only data elements for which the base-10 logarithm of `y` is greater than 2.


### fold

Collapse ("fold") one or more data properties into two properties: a key property (containing the original data property name) and a value property (containing the data value). The __fold__ transform is useful for mapping matrix or cross-tabulation data into a standardized format.

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| fields              | Array&lt;Field&gt;  | An array of field references indicating the data properties to fold.|


The __fold__ transform returns a new array of data objects, with two additional properties: `key` (an extracted property name), and `value` (an extracted data value). The names of these new properties can be changed by setting the transform's output map. For example, the parameter `"output": {"key": "k", "value": "v"}` causes the properties `k` and `v` to be used instead of `key` and `value`.

## Example

```json
{"type": "fold", "fields": ["gold", "silver"]}
```
This example folds the `gold` and `silver` properties. Given the following input data:
```json
[
  {"country": "USA", "gold": 10, "silver": 20}, 
  {"country": "Canada", "gold": 7, "silver": 26}
]
```
this example will produce the following output:
```json
[
  {"key": "gold", "value": 10, "country": "USA", "gold": 10, "silver": 20},
  {"key": "silver", "value": 20, "country": "USA", "gold": 10, "silver": 20},
  {"key": "gold", "value": 7, "country": "Canada", "gold": 7, "silver": 26},
  {"key": "silver", "value": 26, "country": "Canada", "gold": 7, "silver": 26}
]
```


###  formula

Extends data elements with new values according to a calculation formula.

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| field               | String              | The property name in which to store the computed formula value.|
| expr                | [Expression](Expressions)| A string containing an expression (in JavaScript syntax) for the formula. The [expression language](Expressions) includes the variable `datum`, corresponding to the current data object. 


The __formula__ transform returns the input data set, with each element extended with the computed formula value.

## Examples

```json
{"type": "formula", "field": "logx", "expr": "log(datum.x)/LN10"}
```
This example computes the base-10 logarithm of `x` and stores the result on each datum as the `"logx"` property.

```json
{"type": "formula", "field": "hr", "expr": "hours(datum.date)"}
```
This example extracts the hour of the `date` field, and stores the result on each datum as the `hr` property.


### sort

Sorts the values of a data set (whether it be flat or faceted).

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| by                  | Field, Array&lt;Field&gt; | A list of fields to use as sort criteria. By default, ascending order is assumed. Field names may be prepended with a "-" (minus) character to indicate descending order.|


The __sort__ transform returns the input data set with elements sorted in place.

## Example

```json
{"type": "sort", "by": "-_id"}
```
This example sorts a data set in descending order by the value of the `_id` field.


### zip

Merges two data sets together according to a provided join key. If no join key is provided, the data sets are merged by their indices. In essence, __zip__ extends a data set with values from another data set. When matching by index and the secondary (`with`) data set is shorter than the primary data set, the zip transform will cycle through values.

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| with                | String              | The name of the secondary data set to "zip" with the current, primary data set.|
| as                  | String              | The name of the field in which to store the secondary data set values.|
| key                 | Field               | The field in the primary data set to match against the secondary data set.|
| withKey             | Field               | The field in the secondary data set to match against the primary data set.|
| default             |  *                  | A default value to use if no matching key value is found. If not specified `undefined` is used as the default value. The default value is applied only when key values are specified. It will not be used when matching by index.|


The __zip__ transform extends the primary data set with values from a matching member of the secondary ("`with`") data set, and stores the value from the secondary data in the field specified by the _as_ parameter.

## Example

```json
{
  "type": "zip",
  "key": "id",
  "with": "unemployment",
  "withKey": "key",
  "as": "value",
  "default": null
}
```
This example matches records in the input data with records in the data set named "unemployment", where the values of `id` (primary data) and `key` (secondary data) match. Matching values in the secondary data are added to the primary data in the field named "value".

### Visual Encoding Transforms

Visual encoding transforms can be used to create more advanced visualizations, including layout algorithms and geographic projections.

[force](#-force), [geo](#-geo), [geopath](#-geopath), [link](#-link), [pie](#-pie), [stack](#-stack), [treemap](#-treemap)


### force

Performs force-directed layout for network data. Force-directed layouts treat nodes as charged particles and edges (links among nodes) as springs, and uses a physics simulation to determine node positions. The __force__ transform acts on two data sets: one containing nodes and one containing links. Apply the transform to the node data, and include the name of the link data as a transform parameter.

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| links               | String              | The name of the link (edge) data set. Objects in this data set must have appropriately defined `source` and `target` attributes.|
| size                | Array<Number>       | The dimensions [width, height] of this force layout. Defaults to the width and height of the enclosing data rectangle or group.|
| iterations          | Number              | The number of iterations to run the force directed layout. The default value is 500.|
| charge              | Number, Field       | The strength of the charge each node exerts. If the parameter value is a number, it will be used for all nodes. If the parameter is a field definition, the _charge_ will be determined by the data. The default value is -30. Negative values indicate a repulsive force, positive values an attractive force.|
| linkDistance        | Number, Field       | Determines the length of edges, in pixels. If the parameter value is a number, it will be used for all edges. If the parameter is a field definition, the _linkDistance_ will be determined by the data. The default value is 20.|
| linkStrength        | Number, Field       | Determines the tension of edges (the spring constant). If the parameter value is a number, it will be used for all edges. If the parameter is a field definition, the _linkStrength_ will be determined by the data. The default value is 1.|
| friction            | Number              | The strength of the friction force used to stabilize the layout.|
| theta               | Number              | The theta parameter for the Barnes-Hut algorithm, which is used to compute charge forces between nodes.|
| gravity             | Number              | The strength of the pseudo-gravity force that pulls nodes towards the center of the layout area.|
| alpha               | Number              | A "temperature" parameter that determines how much node positions are adjusted at each step.|


By default, the __force__ transform sets the following values on each node datum:
  * **layout_x** - the *x*-coordinate of the current node position.
  * **layout_y** - the *y*-coordinate of the current node position.
  * **layout_px** - the *x*-coordinate of the previous node position.
  * **layout_py** - the *y*-coordinate of the previous node position.
  * **layout_fixed** - a boolean indicating whether node position is locked.
  * **layout_weight** - the node weight; the number of associated links.
These properties may be renamed by specifying an `output` map. For example, `"output": {"fixed": "f", weight": "w"}`.

## Example

```json
{"type": "force", "links": "edges", "linkDistance": 70, "charge": -100, "iterations": 1000}
```

This example assumes a data set named "`edges`" has already been defined, and has appropriate `source` and `target` attributes that reference the graph nodes.


### geo

Performs a cartographic projection. Given longitude and latitude values, sets corresponding x and y properties for a mark.

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| projection          | String              | The type of cartographic projection to use. Defaults to `"mercator"`. The __geo__ transform accepts any projection supported by the D3 projection plug-in (for example, `albersUsa`, `albers`, `hammer`, `winkel3`, etc).|
| lon                 | Field               | The input longitude values.|
| lat                 | Field               | The input latitude values.|
| center              | Array<Number>       | The center of the projection. The value should be a two-element array of numbers.|
| translate           | Array<Number>       | The translation of the projection. The value should be a two-element array of numbers.|
| scale               | Number              | The scale of the projection.|
| rotate              | Number              | The rotation of the projection.|
| precision           | Number              | The desired precision of the projection.|
| clipAngle           | Number              | The clip angle of the projection.|


By default, the __geo__ transform sets the following values on each datum:
* __layout_x__, __layout_y__

## Example

```json
{
  "type": "geo",
  "lat": "latitude",
  "lon": "longitude",
  "projection": "winkel3",
  "scale": 300,
  "translate": [960, 500]
}
```

This example computes a Winkel3 projection for lat/lon pairs stored in the `latitude` and `longitude` attributes.


### geopath

Creates paths for geographic regions, such as countries, states and counties. Given a GeoJSON Feature data value, produces a corresponding path definition, subject to a specified cartographic projection. The __geopath__ transform is intended for use with the __path__ mark type.

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| field               | Field               | The data field containing GeoJSON Feature data.|
| projection          | String              | The type of cartographic projection to use. Defaults to `"mercator"`. The __geo__ transform accepts any projection supported by the D3 projection plug-in (for example, `albersUsa`, `albers`, `hammer`, `winkel3`, etc).|
| center              | Array<Number>       | The center of the projection. The value should be a two-element array of numbers.|
| translate           | Array<Number>       | The translation of the project. The value should be a two-element array of numbers.|
| scale               | Number              | The scale of the projection.|
| rotate              | Number              | The rotation of the projection.|
| precision           | Number              | The desired precision of the projection.|
| clipAngle           | Number              | The clip angle of the projection.


By default, the __geopath__ transform sets the following values on each datum:
* __layout_path__

## Example

```json
{
  "type": "geopath",
  "field": "data",
  "projection": "winkel3",
  "scale": 300,
  "translate": [960, 500]
}
```

This example creates path definitions using a Winkel3 projection applied to GeoJSON data stored in the `data` attribute of input data elements.


### link

Computes a path definition for connecting nodes within a node-link network or tree diagram.

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| source              | Field               | The data field that references the source node for this link.|
| target              | Field               | The data field that references the target node for this link.|
| shape               | String              | A string describing the path shape to use. One of `"line"` (default), `"curve"`, `"diagonal"`, `"diagonalX"`, or `"diagonalY"`.|
| tension             | Number              | A tension parameter in the range [0,1] for the "tightness" of `"curve"`-shaped links.|


By default, the __link__ transform sets the following values on each datum:
* __layout_path__

## Example

```json
{"type": "link", "shape": "line"}
```
Creates straight-line links.

```json
{"type": "link", "shape": "curve", "tension": 0.15}
```
Creates curved links with a limited amount of curvature (tension = 0.15).


### pie

Computes a pie chart layout. Given a set of data values, sets startAngle and endAngle properties for a mark. The __pie__ encoder is intended for use with the __arc__ mark type.

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| field               | Field               | The data values from this field will be encoded as angular spans. If this property is omitted, all pie slices will have equal spans.|
| startAngle          | Number              | A starting angle, in radians, for angular span calculations (default 0).|
| endAngle            | Number              | An ending angle, in radians, for angular span calculations (default 2&pi;).|
| sort                | Boolean             | If true, will sort the data prior to computing angles.|


By default, the __pie__ transform sets the following values on each datum:
* __layout_start__, __layout_end__, __layout_mid__

## Examples

```json
{"type": "pie", "value": "data.price"}
```

Computes angular widths for pie slices based on the field `data.price`.

```json
{"type": "pie"}
```

Computes angular widths for equal-width pie slices.


###  stack

Computes layout values for stacked graphs, as in stacked bar charts or stream graphs.

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| groupby             | Array&lt;Field&gt;  | A list of fields to partition the data into groups (stacks). When values are stacked vertically, this corresponds to the x-coordinates.| 
| field               | Field               | The data field that determines the thickness or height of each stack.|
| sortby              | Array&lt;Field&gt;  | A list of fields to determine the order of stack layers.| 
| offset              | String              | The baseline offset style. One of `"zero"` (default), `"silhouette"`, `"wiggle"`, or `"expand"`. The `"silhouette"` offset will center the stacks, while "`wiggle`" will attempt to minimize changes in slope to make the graph easier to read. If `"expand"` is chosen, the output values will be in the range [0,1].|


By default, the __stack__ transform sets the following values on each datum:
* __layout_start__,  __layout_end__, __layout_mid__

## Examples

```json
{"type": "stack", "groupby": ["x"], "sortby": ["c"], "field": "y"}
```


###  treemap

Computes a squarified [treemap](http://en.wikipedia.org/wiki/Treemapping) layout. The __treemap__ transform is intended for visualizing hierarchical or faceted data with the __rect__ mark type.

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| field               | Field               | The values to use to determine the area of each leaf-level treemap cell.|
| padding             | Number, Array<Number>| The padding (in pixels) to provide around internal nodes in the treemap. For example, this might be used to create space to label the internal nodes. The padding value can either be a single number or an array of four numbers [top, right, bottom, left]. The default padding is zero pixels.|
| ratio               | Number              | The target aspect ratio for the layout to optimize. The default value is the golden ratio, (1 + sqrt(5))/2 =~ 1.618.|
| round               | Boolean             | If true, treemap cell dimensions will be rounded to integer pixels.|
| size                | Array<Number>       | The dimensions [width, height] of the treemap layout. Defaults to the width and height of the enclosing data rectangle or group.|
| sticky              | Boolean             | If true, repeated runs of the treemap will use cached partition boundaries. This results in smoother transition animations, at the cost of unoptimized aspect ratios. If _sticky_ is used, _do not_ reuse the same treemap encoder instance across data sets.|
| children            | Field               | A data field that represents the children array, `children` by default.| 
| sort                | Array&lt;Field&gt;  | A list of fields to use as sort criteria for sibling nodes. By default, ascending order is assumed. Field names may be prepended with a "-" (minus) character to indicate descending order.|


By default, the treemap transform sets the following values on each datum:
* __layout_x__, __layout_y__, __layout_width__, __layout_height__, __layout_depth__

## Example

```json
{"type": "treemap", "field": "price"}
```

Computes a treemap layout where elements are sized according to the field `price`. This example assumes the input data is hierarchical or has already been suitably faceted.
