> [Wiki](Home) ▸ [[Documentation]] ▸ **Marks**

Marks are the basic visual building block of a visualization. Similar to other mark-based frameworks such as [Protovis](http://protovis.org), marks provide basic shapes whose properties can be set according to backing data. Mark properties can be simple constants or data fields, and [Scales](Scales) can be used to map from data to property values. The basic supported mark types are rectangles (`rect`), plotting symbols (`symbol`), general paths or polygons (`path`), circular arcs (`arc`), filled areas (`area`), lines (`line`), images (`image`) and text labels (`text`).

Each mark supports a set of visual _properties_ which determine the position and appearance of mark instances. Typically one mark instance is generated per input data element; the exceptions are the `line` and `area` mark types, which represent multiple data elements as a contiguous line or area shape.

There are three primary property sets: _enter_, _exit_ and _update_. _Enter_ properties are evaluated when data is processed for the first time and a mark instance is newly added to a scene. Similarly, _exit_ properties are evaluated when the data backing a mark is removed, and so the mark is leaving the visual scene. The _update_ properties are evaluated for all existing (non-exiting) mark instances. To better understand how enter, exit and update sets work, readers may wish to peruse [related D3 tutorials](http://bost.ocks.org/mike/join/). In addition, an optional _hover_ set determines visual properties when the mouse cursor hovers over a mark instance. Upon mouse out, the _update_ set is applied.

There is also a special group mark type (`group`) that can contain other marks, as well as local scale, axis and legend definitions. Groups can be used to create visualizations consisting of grouped or repeated elements; examples include stacked graphs (each stack is a separate group containing a series of data values) and small multiples displays (each plot is contained in its own group). See [Group Marks](Group-Marks) for more.

A mark definition typically looks something like this:
```json
{
  "type": "rect",
  "from": {"data": "table"},
  "properties": {
    "enter": {...},
    "exit": {...},
    "update": {...},
    "hover": {...}
  }
}
```

### Top-Level Mark Properties

| Property      | Type          | Description    |
| :------------ |:-------------:| :------------- |
| type          | String        | The mark type (`rect`, `symbol`, `path`, `arc`, `area`, `line`, `rule`, `image`, `text`, `group`)|
| name          | String        | A unique name for the mark instance (optional). As of Vega v1.4.0, the SVG renderer adds the _unaltered_ name value as CSS class names on the enclosing SVG group (`g`) element for mark instances.|
| description   | String        | An optional description of this mark. Can be used as a comment.|
| from          | Object        | An object describing the data this mark set should visualize. The supported properties are `data` (the name of the data set to use) and `transform` (to provide an array of data transformations to apply). If the `data` property is not defined, the mark will attempt to inherit data from its parent group mark, if any. Otherwise, a default, single element data set is assumed.|
| properties    | Object        | An object containing the property set definitions.|
| key           | String        | A data field to use as a unique key for data binding. When a visualization's data is updated, the key value will be used to match data elements to existing mark instances. Use a key field to enable object constancy for transitions over dynamic data.|
| delay         | ValueRef -> Number | The transition delay, in milliseconds, for mark updates. The delay can be set in conjunction with the backing data (possibly through a scale transform) to provide staggered animations.|
| ease          | String        | The transition easing function for mark updates. The supported easing types are `linear`, `quad`, `cubic`, `sin`, `exp`, `circle`, and `bounce`, plus the modifiers `in`, `out`, `in-out`, and `out-in`. The default is `cubic-in-out`. For more details please see the [D3 ease function documentation](https://github.com/mbostock/d3/wiki/Transitions#wiki-d3_ease).|

### Mark Property Sets

The rest of this page describes the available mark properties in greater detail. All visual mark property definitions are specified as name-value pairs in a property set (such as `update`, `enter`, or `exit`). The name is simply the name of the visual property. The value should be a _ValueRef_ or _Production Rule_, as defined below.

### Value References

A value reference (or _ValueRef_) specifies the value for a given mark property. The value may be a constant or a value in the backing data, and may include the application of a scale transform to either.

| Name          | Type          | Description  |
| :------------ |:-------------:| :------------|
| value         | *             | A constant value. If _field_ is specified, this value is ignored.|
| field         | String, Object| A field (property) from which to pull a data value. The corresponding data set is determined by the mark's _from_ property. Dot notation (`"price.min"`) is used to access nested properties; if a dot character is actually part of the property name, you must escape the dot with a backslash: `"some\.field"`.|

If the _field_ property is an object, the following properties may be used
  
| Property      | Type          | Description    |
| :------------ |:-------------:| :------------- |
| datum         | String, Object| Perform a lookup on the current data value (the default operation when `field` is a string).|
| group         | String, Object| Use a property of the enclosing group mark element (e.g., `width` or `height`).|
| parent        | String, Object| Use a property of the enclosing group mark's datum.|

These properties can be arbitrarily nested in order to perform _indirect_ field lookups. For example, `"field": {"parent": {"datum": "f"}}` will first retrieve the value of the `f` field on the current mark's data object. This value will then be used as the property name to lookup on the enclosing group mark's datum. 

**Advanced Use**

`group` and `parent` properties can be given an optional `level` to access grandparents, and higher ancestors. For example, `"field": {"parent": "f", "level": 2}` will use the value of the `f` field of the grandparent's datum. By default, `level = 1` (i.e., parents).

| Property      | Type          | Description    |
| :------------ |:-------------:| :------------- |
| scale         | String, Object| The name of a scale transform to apply. If the input is an object, it indicates a field value from which to dynamically lookup the scale name and follows the format described above. For example `{"datum": "s"}` will use the value of `s` on the current mark's data as the scale name, whereas `{"parent": "t"}` will use the value of `t` on the current group's data as the scale name.|
| mult          | Number        | A multiplier for the value, equivalent to (mult * value). Multipliers are applied after any scale transformation.|
| offset        | Number        | A simple additive offset to bias the final value, equivalent to (value + offset). Offsets are added _after_ any scale transformation and multipliers.|
| band          | Boolean       | If true, and _scale_ is specified, uses the range band of the scale as the retrieved value. This option is useful for determining widths with an ordinal scale.|

##### Examples

The constant value `5`.
```
{"value": 5}
```

The value of `price`, for the current datum.
```
{"field": "price"}
```

The value of `index` for the current datum, multiplied by 20.
```
{"field": "index", "mult": 20}
```

The result of running the value `0` through the scale named `x`.
```
{"scale": "x", "value": 0}
```

The result of running `price` for the current datum through the scale named `y`.
```
{"scale": "y", "field": "price"}
```

The range band width of the ordinal scale `x`. Note that the scale must be ordinal
```
{"scale": "x", "band": true}
```

The range band width of the ordinal scale `x`, reduced (negative offset) by one pixel.
```
{"scale": "x", "band": true, "offset": -1}
```

### Color References

Typically color values are specified as a single value indicating an RGB color. However, sometimes a designer may wish to target specific color fields or use a different color space. In this case a special Value Reference format can be used. In the following example, we can set the red and blue channels of an RGB color as constants, and determine the green channel from a scale transform.
```
"fill": {
 "r": {"value": 255},
 "g": {"scale": "green", "field": "g"},
 "b": {"value": 0}
}
```

Vega supports the following color spaces

| Name          | Description  |
| :------------ | :------------|
| [RGB](http://en.wikipedia.org/wiki/RGB_color_space)| with properties `"r"`, `"g"`, and `"b"`.|
| [HSL](http://en.wikipedia.org/wiki/HSL_and_HSV)| (hue, saturation, lightness), with properties `"h"`, `"s"`, and `"l"`.|
| [CIE LAB](http://en.wikipedia.org/wiki/Lab_color_space)| with properties `"l"`, `"a"`, and `"b"`. A perceptual color space with distances based on human color judgments. The "L" dimension represents luminance, the "A" dimension represents green-red opposition and the "B" dimension represents blue-yellow opposition.|
| [HCL](https://en.wikipedia.org/wiki/Lab_color_space#Cylindrical_representation:_CIELCh_or_CIEHLC)            | (hue, chroma, lightness) with properties `"h"`, `"c"`, and `"l"`. This is a version of LAB which uses polar coordinates for the AB plane.|

### Templates

For String-type properties (e.g., `text` for text marks), a special `template` property can be used instead of a Value Reference. The specified template string can make sure of handlebars, such as ``{{datum.price}}``, to refer to data fields. `datum`, `parent`, and `group` are available for use within the handlebars, and dot notation may be used to refer to nested properties.

Handlebars also support a number of *filters* for text transformation. Filters are specified within a template variable using a pipe-delimited syntax (`{{property | filter1 | filter2 }}`). Some filters may take one or more arguments (`{{property | filter:arg1,arg2 }}`). The available filters are:

| Name          | Description  |
| :------------ | :------------|
| length        | return the length of a string.|
| lower         | maps a string to lowercase (calls `String.toLowerCase()`).|
| upper         | maps a string to uppercase (calls `String.toUpperCase()`).|
| lower-locale  | maps a string to lowercase (calls `String.toLocaleLowerCase()`).|
| upper-locale  | maps a string to uppercase (calls `String.toLocaleUpperCase()`).|
| trim          | remove whitespace at the beginning and end of a string (calls `String.trim()`).|
| left:n        | returns the _n_ left-most characters of a string.|
| right:n       | returns the _n_ right-most characters of a string.|
| mid:n         | returns the _n_ central characters of a string.|
| slice:a,b     | returns a substring according to `String.slice(a, b)`.|
| pad:n,pos     | pads the string with whitespace using the provided length (_n_) and optional position argument (_pos_).|
| truncate:n,pos| truncates the string with the provided length (_n_) and an optional position argument (_pos_).|
| number:fmt    | formats the string as a number using the provided format (_fmt_) string. The filter uses [D3's formatting utilities](https://github.com/mbostock/d3/wiki/Formatting) and accepts a valid D3 number format pattern.|
| time:fmt      | formats the string as a date/time using the provided format (_fmt_) string. The filter uses [D3's time formatting utilities](https://github.com/mbostock/d3/wiki/Time-Formatting) and accepts a valid D3 time format pattern.|

For example,
```json
"text": {
  "template": "{{datum.name}} was bought on {{parent.stamp|time:'%A'}}."
} 
```

### Production Rules

Visual properties can also be set by evaluating an `if-then-else` style chain of _production rules_. A single `rule` property is specified as an array of _ValueRef_ objects, each of which must contain an additional `predicate` property. This is used to invoke a [predicate](Predicates) definition and supply any necessary arguments. A single ValueRef without a predicate may be specified as the final element within the rule to serve as the `else` condition. 

The visual property is set to the ValueRef corresponding to the first predicate that evaluates to `true` within the rule. If none do, the property is set to the final, predicate-less, ValueRef if one is specified. For example, the following specification sets a mark's fill colour using a production rule:

```json
"fill": {
  "rule": [
    {
      "predicate": {
        "name": "isSelected",
        "id": {"field": "_id"}
      },

      "scale": "c", 
      "field": "species"
    },
    {"value": "grey"}
  ]
}
```

Here, the `isSelected` predicate is invoked with an `id` argument (argument values are ValueRefs too!) and, if it evaluates to true, the fill colour is determined by a scale transform. Otherwise, the mark is filled grey. 

### Shared Visual Properties

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| x                   | ValueRef -> Number  | The first (typically left-most) x-coordinate.|
| x2                  | ValueRef -> Number  | The second (typically right-most) x-coordinate.|
| width               | ValueRef -> Number  | The width of the mark (if supported).|
| y                   | ValueRef -> Number  | The first (typically top-most) y-coordinate.|
| y2                  | ValueRef -> Number  | The second (typically bottom-most) y-coordinate.|
| height              | ValueRef -> Number  | The height of the mark (if supported).|
| opacity             | ValueRef -> Number  | The overall opacity.|
| fill                | ValueRef -> Color   | The fill color.|
| fillOpacity         | ValueRef -> Number  | The fill opacity.|
| stroke              | ValueRef -> Color   | The stroke color.|
| strokeWidth         | ValueRef -> Number  | The stroke width, in pixels.|
| strokeOpacity       | ValueRef -> Number  | The stroke opacity.|
| strokeDash          | ValueRef -> Array   | An array of alternating stroke, space lengths for creating dashed or dotted lines.|
| strokeDashOffset    | ValueRef -> Number  | The offset (in pixels) into which to begin drawing with the stroke dash array.|

For marks involving Cartesian extents (e.g., __rect__ marks), the horizontal dimensions are determined by (in order of precedence) the _x_ and _x2_ properties, the _x_ and _width_ properties, and the _x2_ and _width_ properties. If all three of _x_, _x2_ and _width_ are specified, the _width_ value is ignored. The _y_, _y2_ and _height_ properties are treated similarly. For marks without Cartesian extents (e.g., __path__, __arc__, etc) the same calculations are applied, but are only used to determine the mark's ultimate _x_ and _y_ position. That is, a _width_ property may affect the ultimate position, but otherwise is not visualized.

_Note:_ at the time of writing, the _strokeDash_ and _strokeDashOffset_ use bleeding-edge HTML5 canvas support, which can vary across browsers. Browsers with canvas implementations that do not support line dash or line dash offset will ignore these properties. Firefox only supports _strokeDash_ (through the `mozDash` proprietary canvas extension), not _strokeDashOffset_.

### rect

| Property            | Type                | Description  |
| ------------------- |:-------------------:| :------------|
|                     |                     | (No additional properties.)|

### symbol

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| size                | ValueRef -> Number  | The pixel area of the symbol. For example: in the case of circles, the radius is determined in part by the square root of the size value.|
| shape               | ValueRef -> String  | The symbol shape to use. One of `circle` (default), `square`, `cross`, `diamond`, `triangle-up`, or `triangle-down`|

### path

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| path                | ValueRef -> PathString| A path definition in the form of an SVG Path string.|

### arc

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| innerRadius         | ValueRef -> Number  | The inner radius of the arc, in pixels.|
| outerRadius         | ValueRef -> Number  | The outer radius of the arc, in pixels.|
| startAngle          | ValueRef -> Number  | The start angle of the arc, in radians. A value of `0` indicates up or "north", increasing values proceed clockwise.|
| endAngle            | ValueRef -> Number  | The end angle of the arc, in radians. A value of `0` indicates up or "north", increasing values proceed clockwise.|

### area

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| interpolate         | ValueRef -> String  | The line interpolation method to use. One of `linear`, `step-before`, `step-after`, `basis`, `basis-open`, `cardinal`, `cardinal-open`, `monotone`.|
| tension             | ValueRef -> Number  | Depending on the interpolation type, sets the tension parameter.|

### line

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| interpolate         | ValueRef -> String  | The line interpolation method to use. One of `linear`, `step-before`, `step-after`, `basis`, `basis-open`, `basis-closed`, `bundle`, `cardinal`, `cardinal-open`, `cardinal-closed`, `monotone`.|
| tension             | ValueRef -> Number  | Depending on the interpolation type, sets the tension parameter.|

### image

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| url                 | ValueRef -> String  | The URL from which to retrieve the image.|
| align               | ValueRef -> String  | The horizontal alignment of the image. One of `left`, `right`, `center`.|
| baseline            | ValueRef -> String  | The vertical alignment of the image. One of `top`, `middle`, `bottom`.|

### text

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| text                | ValueRef -> String  | The text to display.|
| align               | ValueRef -> String  | The horizontal alignment of the text. One of `left`, `right`, `center`.|
| baseline            | ValueRef -> String  | The vertical alignment of the text. One of `top`, `middle`, `bottom`.|
| dx                  | ValueRef -> Number  | The horizontal margin, in pixels, between the text label and its anchor point. The value is ignored if the _align_ property is `center`.|
| dy                  | ValueRef -> Number  | The vertical margin, in pixels, between the text label and its anchor point. The value is ignored if the _baseline_ property is `middle`.|
| radius              | ValueRef -> Number  | Polar coordinate radial offset, in pixels, of the text label from the origin determined by the `x` and `y` properties.|
| theta               | ValueRef -> Number  | Polar coordinate angle, in radians, of the text label from the origin determined by the `x` and `y` properties. Values for `theta` follow the same convention of `arc` mark `startAngle` and `endAngle` properties: angles are measured in radians, with `0` indicating "north".|
| angle               | ValueRef -> Number  | The rotation angle of the text, in degrees.|
| font                | ValueRef -> String  | The typeface to set the text in (e.g., `Helvetica Neue`).|
| fontSize            | ValueRef -> Number  | The font size, in pixels.|
| fontWeight          | ValueRef -> String  | The font weight (e.g., `bold`).|
| fontStyle           | ValueRef -> String  | The font style (e.g., `italic`).|

### group

Group marks have the same visual properties as __rect__ marks. They can also contain children marks, as well as scales, axes and legends. See the [Group Marks](Group-Marks) page for more details.
