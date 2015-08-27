> [Wiki](Home) ▸ [[Documentation]] ▸ **Scales**

Scales are functions that transform a _domain_ of data values (numbers, dates, strings, etc) to a _range_ of visual values (pixels, colors, sizes). A scale function takes a single data value as input and returns a visual value. Vega includes different types of scales for quantitative data or ordinal/categorical data.

### Common Scale Properties

| Property      | Type          | Description    |
| :------------ |:-------------:| :------------- |
| name          | String        | A unique name for the scale.|
| type          | String        | The type of scale. If unspecified, the default value is `linear`. For ordinal scales, the value should be `ordinal`. For dates and times the value should be `time` or `utc` (for UTC time). The supported quantitative scale types  are `linear`, `log`, `pow`, `sqrt`, `quantile`, `quantize`, and `threshold`. |
| domain        | Array  [DataRef](#scale-domains)    | The domain of the scale, representing the set of data values. For quantitative data, this can take the form of a two-element array with minimum and maximum values. For ordinal/categorical data, this may be an array of valid input values. The domain may also be specified by a reference to a data source.|
| domainMin     | Number  [DataRef](#scale-domains)   |  For quantitative scales only, sets the minimum value in the scale domain. domainMin can be used to override, or (with domainMax) used in lieu of, the domain property.|
| domainMax     | Number  [DataRef](#scale-domains)   | For quantitative scales only, sets the maximum value in the scale domain. domainMax can be used to override, or (with domainMin) used in lieu of, the domain property.|
| range         | Array  String  [DataRef](#scale-domains)  | The range of the scale, representing the set of visual values. For numeric values, the range can take the form of a two-element array with minimum and maximum values. For ordinal or quantized data, the range may by an array of desired output values, which are mapped to elements in the specified domain. See the section on range literals below for more options. For _ordinal scales only_, the range can be defined using a [DataRef](#scale-domains): the range values are then drawn dynamically from a backing data set.|
| rangeMin      | *              | Sets the minimum value in the scale range. rangeMin can be used to override, or (with rangeMax) used in lieu of, the range property.|
| rangeMax      | *              | Sets the maximum value in the scale range. rangeMax can be used to override, or (with rangeMin) used in lieu of, the range property.|
| reverse       | Boolean        | If true, flips the scale range.|
| round         | Boolean        | If true, rounds numeric output values to integers. This can be helpful for snapping to the pixel grid.|

### Ordinal Scale Properties

| Property      | Type          | Description    |
| :------------ |:-------------:| :------------- |
| points        | Boolean       | If true, distributes the ordinal values over a quantitative range at uniformly spaced points. The spacing of the points can be adjusted using the _padding_ property. If false, the ordinal scale will construct evenly-spaced bands, rather than points.|
| padding       | Number        | Applies spacing among ordinal elements in the scale range. The actual effect depends on how the scale is configured. If the __points__ parameter is `true`, the padding value is interpreted as a multiple of the spacing between points. A reasonable value is 1.0, such that the first and last point will be offset from the minimum and maximum value by half the distance between points. Otherwise, padding is typically in the range [0, 1] and corresponds to the fraction of space in the range interval to allocate to padding. A value of 0.5 means that the range band width will be equal to the padding width. For more, see the [D3 ordinal scale documentation](https://github.com/mbostock/d3/wiki/Ordinal-Scales).|
| sort          | Object        | If set, the values in the scale domain will be sorted based on an aggregate calculation over a specified sort field.|
| sort field    | Field         | The field name to aggregate over.|
| sort op       | String        | A valid [aggregation operation](Data-Transforms#-aggregate) (e.g., `mean`, `median`, etc.)|
| sort order    | String        | Either `asc` or `desc` for ascending or descending, respectively.|

### Time Scale Properties

| Property      | Type          | Description    |
| :------------ |:-------------:| :------------- |
| clamp         | Boolean       | If true, values that exceed the data domain are clamped to either the minimum or maximum range value.|
| nice          | String        | If specified, modifies the scale domain to use a more human-friendly value range. For `time` and `utc` scale types only, the nice value should be a string indicating the desired time interval; legal values are `"second"`, `"minute"`, `"hour"`, `"day"`, `"week"`, `"month"`, or `"year"`.|

### Quantitative Scale Properties

| Property      | Type          | Description    |
| :------------ |:-------------:| :------------- |
| clamp         | Boolean       | If true, values that exceed the data domain are clamped to either the minimum or maximum range value.|
| exponent      | Number        | Sets the exponent of the scale transformation. For `pow` scale types only, otherwise ignored.|
| nice          | Boolean       | If true, modifies the scale domain to use a more human-friendly number range (e.g., 7 instead of 6.96).|
| zero          | Boolean       | If true, ensures that a zero baseline value is included in the scale domain. This option is ignored for non-quantitative scales.|

### Scale Domains

Scale domains may be defined directly as an array of values, or can be inferred from input data. In the latter case, the scale domain can be defined in Vega as an object we call a "DataRef" (for _data reference_). In most cases, a DataRef is simply an object with up to two properties:

| Property      | Type          | Description    |
| :------------ |:-------------:| :------------- |
| data          | String        | The name of the data set containing domain values.|
| field         | Field  Array&lt;Field&gt;  Object  Array&lt;Object&gt; | A reference to the desired data field(s) (e.g., `"price"`). An array of fields will include values for all referenced fields. Typically string values are used to indicate the field to lookup. However, you can specify an indirect lookup of the field name using an object parameter of the form `{"parent": "f"}`. In this case, the value of the field `f` is first retrieved from the enclosing group's datum, and then used as the field name for the current data set. In other words, you can determine the field name dynamically from your data.|

**Advanced Use**

For scales that are defined within a __group__ mark, you can omit the _data_ property to tell Vega to instead use the data values that are bound to the __group__ mark instance.

### Scale Domains Drawn From Multiple Fields or Data Sets

To compute a scale domain over multiple data properties, you would typically use an array of field references. However, sometimes you may need to compute the domain over multiple fields from _different data sets_. For this case, there is a special form of DataRef: an object with a single property ( __fields__ ) that contains an array of DataRef instances.

Here is an example that constructs a domain using the fields `price` and `cost` drawn from two different data sets:
```json
"domain": {
  "fields": [
    {"data": "table1", "field": "price"},
    {"data": "table2", "field": "cost"}
   ]
}
```

**For sorted ordinal scales**

Each `fields` object may contain an additional `sort` property to specify different sort fields for each data set. When set, this property will be used instead of the `sort.field` property in the ordinal scale definition.

### Scale Range Literals

The following string values can be used to automatically set a scale range.

| Name          | Description  |
| :------------ | :------------|
| width         | Set the scale range to `[0, width]`, where the width value is defined by the enclosing group or data rectangle.|
| height        | Set the scale range to `[0, height]`, where the height value is defined by the enclosing group or data rectangle.|
| shapes        | Set the scale range to the symbol type names: `["circle", "cross", "diamond", "square", "triangle-down", "triangle-up"]`|
| category10    | Set the scale range to a 10-color categorical palette. |
| category20    | Set the scale range to a 20-color categorical palette. |

