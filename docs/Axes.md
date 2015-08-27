> [Wiki](Home) ▸ [[Documentation]] ▸ **Axes**

Axes provide axis lines, ticks and labels to convey how a spatial range represents a data range. Simply put, axes visualize scales. Vega currently supports axes for Cartesian (rectangular) coordinates. Future versions may introduce support for polar (circular) coordinates. Similar to scales, axes can be defined either at the top-level visualization, or as part of a __group__ mark.

Axes provide three types of tick marks: major, minor and end ticks. Minor ticks use smaller lines than major ticks. End ticks appear at the edges of the scales.

### Axis Properties

| Property      | Type          | Description    |
| :------------ |:-------------:| :------------- |
| type          | String        | The type of axis. One of `x` or `y`.|
| scale         | String        | The name of the scale backing the axis component.|
| orient        | String        | The orientation of the axis. One of `top`, `bottom`, `left` or `right`. The orientation can be used to further specialize the axis type (e.g., a `y` axis oriented for the `right` edge of the chart).|
| title         | String        | A title for the axis (none by default).|
| titleOffset   | Number        | The offset (in pixels) from the axis at which to place the title.|
| format        | String        | The formatting pattern for axis labels. Vega uses [D3's format pattern](https://github.com/mbostock/d3/wiki/Formatting).|
| ticks         | Number        | A desired number of ticks, for axes visualizing quantitative scales. The resulting number may be different so that values are "nice" (multiples of 2, 5, 10) and lie within the underlying scale's range.|
| values        | Array         | Explicitly set the visible axis tick values.|
| subdivide     | Number        | If provided, sets the number of minor ticks between major ticks (the value 9 results in decimal subdivision). Only applicable for axes visualizing quantitative scales.| 
| tickPadding   | Number        | The padding, in pixels, between ticks and text labels.|
| tickSize      | Number        | The size, in pixels, of major, minor and end ticks.|
| tickSizeMajor | Number        | The size, in pixels, of major ticks.|
| tickSizeMinor | Number        | The size, in pixels, of minor ticks.|
| tickSizeEnd   | Number        | The size, in pixels, of end ticks.|
| offset        | Number, Object| If a number, then the value is the offset, in pixels, by which to displace the axis from the edge of the enclosing group or data rectangle. The offset can also be specified as an object with `scale` and `value` properties in which `scale` refers to the name of a scale and `value` is a value in the domain of the scale. The resulting value will be a number in the range of the scale.|
| layer         | String        | A string indicating if the axis (and any gridlines) should be placed above or below the data marks. One of `"front"` (default) or `"back"`.|
| grid          | Boolean       | A flag indicate if gridlines should be created in addition to ticks.|
| properties    | Object        | Optional mark property definitions for custom axis styling. The input object can include sub-objects for `ticks` (both major and minor), `majorTicks`, `minorTicks`, `labels` and `axis` (for the axis line).|

### Custom Axis Styles

Custom mark properties can be set for all axis elements through the axis __properties__ setting. The addressable elements are `ticks` (both major and minor), `majorTicks`, `minorTicks`, `grid` (for gridlines), `labels`, `title` and `axis` (for the axis line, including end ticks). Each element can be styled using standard Vega _Value References_, as described in the [Marks](Marks) documentation.

Note that enclosing ticks at the start and end of the axis are drawn as part of the axis baseline. Custom styling properties for `axis` will thus affect both the axis line and the end ticks, if shown.

The following example shows how to set custom colors, thickness, text angle, and fonts.
```json
"axes": [
   {
     "type": "x",
     "scale": "x",
     "title": "X-Axis",
     "properties": {
       "ticks": {
         "stroke": {"value": "steelblue"}
       },
       "majorTicks": {
         "strokeWidth": {"value": 2}
       },
       "labels": {
         "fill": {"value": "steelblue"},
         "angle": {"value": 50},
         "fontSize": {"value": 14},
         "align": {"value": "left"},
         "baseline": {"value": "middle"},
         "dx": {"value": 3}
       },
       "title": {
         "fontSize": {"value": 16}
       },
       "axis": {
         "stroke": {"value": "#333"},
         "strokeWidth": {"value": 1.5}
       }
     }
   }
]
```

Custom text can be defined using the `"text"` property for `labels`. For example, one could define an ordinal scale that serves as a lookup table from axis values to axis label text.
