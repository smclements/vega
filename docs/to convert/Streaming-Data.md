> [Wiki](Home) ▸ [[Documentation]] ▸ **Streaming Data**

Vega 2.0 supports streaming data through a set of operators that can be used within the specification, and API endpoints on the [View component](#view-component-streaming-api). 

## Streaming Operators

Within a dataset definition, a pipeline of streaming operators can be defined under the __modify__ property. These operators take the following properties:

* __type__ [String] - Either `insert`, `remove`, or `toggle`.
* __signal__ [String] - A signal name, with dot notation to reference nested properties. The value of this signal is inserted, removed, or toggled from the dataset.
* __field__ [String] - A field name under which the signal value is inserted. When removing or toggling signal values, this field name is used to find existing matches within the dataset. 

Streaming operators are only evaluated when a signal value changes. They run _after_ all data transformations for the corresponding data set have been executed. In the event that multiple operators use the same signal, operators are evaluated in their definition order.

A special `clear` operator is also available, and takes a `predicate` property. This operator clears the dataset whenever the given predicate evaluates to true. 

Here is an excerpt of the shift-click example that demonstrates the use of streaming operators in the specification. Clicking a point selects it, and multiple points can be selected when the shift key is pressed. Data values of the form `{"point_id": 5}` are added and removed from the `selectedPts` dataset based on user interaction.

```json
{
  "signals": [
    {
      "name": "clickedPt",
      "streams": [{"type": "click", "expr": "datum._id"}]
    },
    {
      "name": "shift",
      "streams": [{"type": "click", "expr": "event.shiftKey"}]
    }
  ],

  "data": [
    {
      "name": "selectedPts",
      "modify": [
        {
          "type": "clear",
          "predicate": "clearPts"
        },
        {
          "type": "toggle",
          "signal": "clickedPt",
          "field": "point_id"
        }
      ]
    }
  ]
}
```

## View Component Streaming API

Parsing a Vega specification, and rendering a visualization, produces a [View Component](Runtime) which exposes an [API](Runtime#view-component-api). The following API methods are available to support streaming data externally:

* view<b>.data(name)</b> - Returns a streaming API for the name dataset. The following methods are available, and can be invoked using a method chaining style:
  * data<b>.insert(values)</b> - Inserts the given array of data values into the dataset.
  * data<b>.update(where, field, modify)</b> - Updates the value of _field_ for all data values that match the _where_ condition. _where_ is a function that is called with each data value, and must return `true` or `false`. _modify_ is a function that is called with each _matching_ data value, and must return the new _field_ value. 
  * data<b>.remove(where)</b> - Removes all data values that match the _where_ condition -- a function that is called for each data value, and must return `true` or `false`.
  * data<b>.values()</b> - Returns all values currently in the dataset.

For convenience, an object can also be passed to the data API method where keys correspond to dataset names, and values are functions that receive the corresponding dataset's streaming API. 

Once a dataset has been modified, a __view.update__ call must be executed in order to update the visualization. For example,

```js
view.data('table')
  .insert([{"x": 1}, {"x": 2}])
  .update(function(d) { return d.x > 5; }, 'y', function(d) { return d.y + 5; });

view.data({
  table2: function(data) {
    data.remove(function(d) { return d.x > 5; });
  },
  table3: function(data) {
    data.insert([{"x": 4}, {"x": 5}]);
  }
});

view.update();
```