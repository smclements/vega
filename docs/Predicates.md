> [Wiki](Home) ▸ [[Documentation]] ▸ **Predicates**

Predicates define conditions, and are used with [production rules](Marks#production-rules) to describe the visual properties of marks. They are particularly useful for allowing users to select and manipulate data and marks interactively. 

### Common Predicate Properties

| Property      | Type          | Description    |
| :------------ |:-------------:| :------------- |
| name          | String        | A unique name for the predicate. |
| type          | String        | The type of predicate. Supported types include comparisons (`==`, `!=`, `>`, `<`, `>=`, `<=`), boolean logic (`and`, `or`), and range/set inclusion (`in`).|
| operands      | Array<Objects>| Except for `in`-type predicates, lists the operands used to determine the predicate's `true`-condition. Each operand is an object with one of the following properties:|
| _value_       | *             | A constant value.|
| _signal_      | String        | A signal name, and dot notation can be used to access nested signal values.| 
| _arg_         | String        | The name of an argument, to allow this operand's value to be specified when invoking the predicate.|
| _predicate_   | Object        | An object that specifies another predicate's `name` and key/value pairs of any arguments it takes.| 

Predicates return `true` if the specified operands meet the condition as specified by the predicate type. Consider the following example,
```json
{
  "predicates": [
    {
      "name": "inStock",
      "type": ">",
      "operands": [{"signal": "amount"}, {"value": 0}]
    },
    {
      "name": "discount",
      "type": "==",
      "operands": [{"signal": "price"}, {"arg": "dollars"}]
    },
    {
      "name": "predicateC",
      "type": "and",
      "operands": [
        {"predicate": "inStock"}, 
        {"predicate": {"name": "discount", "dollars": {"value": 5}}}
      ]
    }
  ]
}
```
The `inStock` predicate evaluates to true whenever the value of the `amount` signal is greater than `0`. Similarly, the `discount` predicate is true whenever the `price` signal is equal to the value passed as the `dollars` argument, as shown in `predicateC`. 

If `predicateC` did not specify a `dollars` argument, then one would need to be provided when invoking `predicateC` later in the specification (i.e., argument names cascade).

### In-Predicate Properties

In-predicates test whether a specified item exists within a range or set (e.g., a data set or ordinal scale's domain). Besides the _name_ and _type_ properties described above, in-predicates can take the following properties:

| Property      | Type          | Description    |
| :------------ |:-------------:| :------------- |
| item          | String        | A value, signal, or argument operand as described above.|  
| data          | String        | The name of a data set.|
| field         | String        | If _data_ is specified, the name of a field. The predicate returns true if there exists a record in the data set, where the value of given field matches _item_.| 
| range         | Array<Operands>| An array with two operands. The predicate returns true if _item_ lies within the range.|
| scale         | [ScopedScaleRef](Signals#scoped-scale-reference)  | A scale transform to apply to the range. By using an inverse scale transform, the in-predicate can be defined over a range of data values, rather than visual or pixel values.| 

For example, 
```json
{
  "predicates": [
    {
      "name": "xRange",
      "type": "in",
      "item": {"arg": "x"},
      "range": [{"signal": "brush_start.x"}, {"signal": "brush_end.x"}],
      "scale": {"name": "x", "invert": true}
    },
    {
      "name": "isSelected",
      "type": "in",
      "item": {"arg": "id"},
      "data": "selectedPts",
      "field": "_id"
    }
  ]
}
```

The `xRange` predicate tests whether a given `x` argument falls within a range of data values whose pixel values, after an x-scale transform, lie within the `brush_start` and `brush_end` signals. Similarly, the `isSelected` predicate tests whether the `_id` field of a record in the `selectedPts` data set contains the value of the `id` argument.
