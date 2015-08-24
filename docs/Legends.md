## LEGENDS


Similar to axes, legends visualize scales. However, whereas axes aid interpretation of scales with spatial ranges, legends aid interpretation of scales with ranges such as colors, shapes and sizes. Similar to scales and axes, legends can be defined either at the top-level visualization, or as part of a __group__ mark.

Legends take one or more scales as their primary input. At least one of the __size__, __shape__, __fill__ or __stroke__ parameters must be specified. If multiple scales are provided, it is very important that they share the _same domain_ of input vales. Otherwise, the behavior of the legend may be unpredictable.

### Legend Properties

| Property      | Type          | Description    |
| :------------ |:-------------:| :------------- |
| size          | String        | The name of the scale that determines an item's size.|
| shape         | String        | The name of the scale that determines an item's shape.|
| fill          | String        | The name of the scale that determines an item's fill color.|
| stroke        | String        | The name of the scale that determines an item's stroke color.|
| orient        | String        | The orientation of the legend. One of `"left"` or `"right"`. This determines how the legend is positioned within the scene. The default is `"right"`.|
| title         | String        | The title for the legend (none by default).|
| format        | String        | An optional formatting pattern for legend labels. Vega uses [D3's format pattern](https://github.com/mbostock/d3/wiki/Formatting).|
| values        | Array         | Explicitly set the visible legend values.|
| properties    | Object        | Optional mark property definitions for custom legend styling. The input object can include sub-objects for `title`, `labels`, `symbols` (for discrete legend items), `gradient` (for a continuous color gradient) and `legend` (for the overall legend group).|

### Custom Legend Styles
Custom mark properties can be set for all legend elements through the legend __properties__ setting. The addressable elements are the `title`, `labels`, `symbols` (for discrete legend items), `gradient` (for a continuous color gradient) and `legend` (for the overall legend group). Each element can be styled using standard Vega _Value References_, as described in the [Marks](#marks) documentation.

The following example shows how to set custom fonts and a border on a legend for a fill color encoding.
```json
"legends": [
   {
     "fill": "color",
     "properties": {
       "title": {
         "fontSize": {"value": 14}
       },
       "labels": {
         "fontSize": {"value": 12}
       },
       "symbols": {
         "stroke": {"value": "transparent"}
       },
       "legend": {
         "stroke": {"value": "#ccc"},
         "strokeWidth": {"value": 1.5}
       }
     }
   }
]
```

Custom text can be defined using the `"text"` property for `labels`. For example, one could define an ordinal scale that serves as a lookup table from axis values to axis label text.

In addition, one can set the `"x"` and `"y"` properties for the `legend` to perform custom positioning, overriding automatic legend positioning.


-----