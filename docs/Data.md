> [Wiki](Home) ▸ [[Documentation]] ▸ **Data**

The basic data model used by Vega is _tabular_ data, similar to a spreadsheet or database table. Individual data sets are assumed to contain a collection of records (or "rows"), which may contain any number of named data attributes (fields, or "columns"). Upon load, Vega maps each data record into a data object and assigns it a unique `_id`. 

For example, if a Vega spec loads input JSON data like this:
```json
[{"x":0, "y":3}, {"x":1, "y":5}]
```
the input data is then loaded into data objects like this:
```json
[{"_id":0, "x":0, "y":3}, {"_id":1, "x":1, "y":5}]
```

Data sets can be specified directly (either through including the data inline or providing a URL from which to load the data), or bound dynamically at runtime (by providing data at chart instantiation time). Note that loading data from a URL will be subject to the policies of your runtime environment (e.g., cross-domain request rules).

##### Examples

Here is an example defining data directly in a specification:
```json
{"name": "table", "values": [12, 23, 47, 6, 52, 19]}
```

One can also load data from an external file (in this case, JSON):
```json
{"name": "points", "url": "data/points.json"}
```

Or, one can simply declare the existence of a data set. The data can then be dynamically provided when the visualization is instantiated (see the [Runtime](Runtime) documentation for more):
```json
{"name": "table"}
```

Finally, one might copy an existing data set and/or apply data transforms. In this case, we create a new data set ("stats") by computing statistics for data groups (facets) computed from the "table" data set:
```json
{
  "name": "stats",
  "source": "table",
  "transform": [
    {
      "type": "facet", 
      "groupby": ["x"], 
      "summarize": {"y": ["average", "sum", "min", "max"]}
    }
  ]
}
```

### Data Properties
| Property      | Type          | Description    |
| :------------ |:-------------:| :------------- |
| name          | String        | A unique name for the data set. |
| format        | Object        | An object that specifies the format for the data file, if loaded from a URL. The currently supported formats are `json` (JavaScript Object Notation), `csv` (comma-separated values), `tsv` (tab-separated values), `topojson`, and `treejson`. These options are specified by the `type` property of the _format_ object. For other parameters, see the Formats documentation below. |
| values        | JSON          |  The actual data set to use. The _values_ property allows data to be inlined directly within the specification itself. |
| source        | String        | The name of another data set to use as the source for this data set. The _source_ property is particularly useful in combination with a transform pipeline to derive new data. |
| url           | String        | A URL from which to load the data set. Use the _format_ property to ensure the loaded data is correctly parsed. If the _format_ property is not specified, the data is assumed to be in a row-oriented JSON format. |
| transform     | Array&lt;Transform&gt;     | An array of transforms to perform on the data. Transform operators will be run on the default data, as provided by late-binding or as specified by the _source_, _values_, or _url_ properties. See the [Data Transforms](Data-Transforms) documentation for more details. |
| modify        | Array&lt;StreamingOps&gt;  | An array of streaming operators to insert, remove, &amp; toggle data values, or clear the data set entirely. These operators are run _after_ data transforms. See the [Streaming Data](Streaming-Data) documentation for more details. |


### Formats

The supported formats for loading data from external files are:

###  json

Loads a JavaScript Object Notation (JSON) file. Assumes row-oriented data, where each row is an object with named attributes. This is the default file format, and so will be used if no `format` parameter is provided. If specified, the `format` parameter should have a `type` property of `"json"`, and can also accept the following:

| Name          | Type          | Description    |
| :------------ |:-------------:| :------------- |
| parse         | Object        | A collection of parsing instructions that can be used to define the data types of string-valued attributes in the JSON file. Each instruction is a name-value pair, where the name is the name of the attribute, and the value is the desired data type (one of `"number"`, `"boolean"` or `"date"`). For example, `"parse": {"modified_on":"date"}` ensures that the `modified_on` value in each row of the input data is parsed as a Date value. |
| property      | String        | The JSON property containing the desired data. This parameter can be used when the loaded JSON file may have surrounding structure or meta-data. For example `"property": "values.features"` is equivalent to retrieving `json.values.features` from the loaded JSON object. |

###  csv

Load a comma-separated values (CSV) file. The properties of the loaded JSON object are taken from the values of the first row of the file. The `format` parameter should have a `type` property of `"csv"`, and can also accept the following:

| Name          | Type          | Description  |
| :------------ |:-------------:| :------------|
| parse         | Object        | A collection of parsing instructions that can be used to define the data types of attributes in the CSV file. By default, all attributes are treated as string-typed data. Each instruction is a name-value pair, where the name is the name of the attribute, and the value is the desired data type (one of `"number"`, `"boolean"` or `"date"`). For example, `"parse": {"y":"number"}` ensures that the `y` value in each row of the input data is parsed as a numerical value. |

###  tsv

Load a tab-separated values (TSV) file. The properties of the loaded JSON object are taken from the values of the first row of the file. The `format` parameter should have a `type` property of `"tsv"`, and can also accept the following:

| Name          | Type          | Description  |
| :------------ |:-------------:| :------------|
| parse         | Object        | A collection of parsing instructions that can be used to define the data types of attributes in the TSV file. By default, all attributes are treated as string-typed data. Each instruction is a name-value pair, where the name is the name of the attribute, and the value is the desired data type (one of `"number"`, `"boolean"` or `"date"`). For example, `"parse": {"y":"number"}` ensures that the `y` value in each row of the input data is parsed as a numerical value. |

###  topojson

Load a JavaScript Object Notation (JSON) file using the [TopoJSON](https://github.com/mbostock/topojson/wiki) format. The input file must contain valid TopoJSON data. The TopoJSON input is then converted into a GeoJSON format for use within Vega. There are two mutually exclusive properties that can be used to specify the conversion process:

| Name          | Type          | Description  |
| :------------ |:-------------:| :------------|
| feature       | String        | The name of the TopoJSON object set to convert to a GeoJSON feature collection. For example, in a map of the world, there may be an object set named `"countries"`. Using the feature property, we can extract this set and generate a GeoJSON feature object for each country. |
| mesh          | String        | The name of the TopoJSON object set to convert to a mesh. Similar to the __feature__ option, __mesh__ extracts a named TopoJSON object set. Unlike the __feature__ option, the corresponding geo data is returned as a single, unified mesh instance, not as inidividual GeoJSON features. Extracting a mesh is useful for more efficiently drawing borders or other geographic elements that you do not need to associate with specific regions such as individual countries, states or counties. |

###  treejson

Load a JavaScript Object Notation (JSON) file that contains hierarchical (tree) data. This format consists of a top-level JSON object (the root) that includes a named array of child objects. Each child may then have its own children in turn. For an example of this format, see [flare.json](http://vega.github.io/vega/data/flare.json). The treejson reader accepts the following properties:

| Name          | Type          | Description  |
| :------------ |:-------------:| :------------|
| children      | String        | The JSON property that contains an array of children nodes for each intermediate node. This parameter defaults to `"children"`.|
| parse         | Object        | A collection of parsing instructions that can be used to define the data types of string-valued attributes in the JSON file. Each instruction is a name-value pair, where the name is the name of the attribute, and the value is the desired data type (one of `"number"`, `"boolean"` or `"date"`). For example, `"parse": {"modified_on":"date"}` ensures that the `modified_on` value in each row of the input data is parsed as a Date value.|

### Transforms

Data sets can also be manipulated by a number of transforms. Transformations are specified as an array of transform definitions. See the [Data Transforms](Data-Transforms) documentation for more details.

### Streaming Data

Vega 2.0 supports streaming inserts, updates, and removals of data values. Streaming operations can be specified directly within the specification, or can be executed via an API for external updates. See the [Streaming Data](Streaming-Data) documentation for more details.
