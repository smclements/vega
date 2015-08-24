## SIGNALS

Signals are dynamic variables that drive interactive behaviours. They can be used throughout a Vega specification (e.g., with mark or data transform properties), and their values are determined by expressions or event streams. Event streams capture and sequence hardware events (e.g., `mousedown` or `touchmove`). When an event occurs, dependent signals are re-evaluated in their specification order. These new signal values propagate to the rest of the specification, and the visualization is re-rendered automatically.

A signal definition, and it's use in the rest of a specification, looks something like this:

```json
{
  "signals": [{
    "name": "indexDate",
    "streams": [{
      "type": "mousemove", 
      "expr": "eventX()", 
      "scale": {"name": "x", "invert": true}
    }]
  }],

  "data": [{
    "name": "index",
    "source": "stocks",
    "transform": [
      {
        "type": "filter",
        "test": "month(datum.date) == month(indexDate)"
      }
    ]
  }],

  "marks": [{
    "type": "rule",
    "properties": {
      "update": {
        "x": {"scale": "x", "signal": "indexDate"}
      }
    } 
  }]
}
```

### Signal Properties

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| name                | String              | A unique name for the signal. Reserved keywords (including `datum`, `event`, `signals` and [function names](Expressions)) may not be used.| 
| init                | Object, or *        | The initial value of the signal. If `init` is an object with an `expr` property, the expression is evaluated to produce the signal's initial value. An additional `scale` property can also be specified to invoke a [Scoped Scale Reference](#scoped-scale-references).|

### Expression Values

Signal values can be determined in one of two ways: purely by other signals, or by event streams as well. If a signal is only dependent on other signals, two additional properties can be specified:

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| expr                | [Expression](Expressions)| A string containing an expression (in JavaScript syntax) for the signal value. It is automatically reevaluated whenever used signal values change.| 
| scale               | [ScopedScaleRef](#scoped-scale-references) | A scale transform to apply to the expression. This can be particularly useful for inverting expression values (i.e., moving them from visual/pixel space to data space).|

### Event Stream Values

To have interaction events (e.g., `mousedown`, or `touchmove`) trigger changes in signal values, an additional __streams__ property must be defined as an array of objects with the following properties:

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| type                | [EventSelector](#event-stream-selectors)| A string that uses the [event selector syntax](#event-stream-selectors).| 
| expr                | [Expression](Expressions)               | A string containing an expression (in JavaScript syntax) that is reevaluated every time the specified events occur. `event` and `datum` variables are available for use here, corresponding to the captured DOM event and the data object backing the event's target item.|

For events that occur within the visualization, the following special event functions are also available

| Event               | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| eventX              | Number              | An x-coordinate relative to the visualization container element.|
| eventY              | Number              | A y-coordinate relative to the visualization container element.|
| eventXY             | Number              | Combining the above two functions into an object with `x` and `y` properties.|
| eventItem           | Item                | The event target item within the Vega scenegraph.| 
| eventGroup          | Item                | The target's enclosing group mark item within the Vega scenegraph.|
| scale               | [ScopedScaleRef](#scoped-scale-references)| A scale transform to apply to the expression. This can be particularly useful for inverting expression values (i.e., moving them from visual/pixel space to data space).|

`eventGroup` takes an optional _name_ argument to return a named ancestor group mark item. Only named ancestors of the event target are addressable here. 

Similarly, the `eventX`, `eventY`, and `eventXY` take an optional argument that can either be the _name_ or scenegraph _item_ of an enclosing ancestor. If provided, coordinates are translated to the given group's coordinates. 

For example, if a rectangle mark item in the following specification was clicked
```json
{
  "marks": [{
    "name": "foo",
    "type": "group",

    "marks": [{
      "name": "bar",
      "type": "group",

      "marks": [{
        "type": "rect"
      }]
    }]
  }]
}
```
`eventItem` would return the specific rectangle that was clicked. `eventGroup()` would return the enclosing group mark item (`bar`), and `eventGroup('foo')` would return its parent. `eventX` and the others would return the position of the click relative to the entire visualization, whereas `eventX('foo')` would return the position of the click within the `foo` group mark. 


### Event Stream Selectors

Event selectors specify the sequence of events ("event stream") that must occur in order to trigger an interactive behaviour. The syntax consists of the following:

| Name                | Description            |
| ------------------- | ---------------------- |
| eventType           | Captures events of a specific type, for example `mousedown`, or `touchmove`. By default, this captures all events of the given type that occur anywhere on the visualization.|
| <b>target:</b>eventType | Filters for only events that occur on the given target. The following targets are recognized |
| * markType              | Filters for events that occur on mark instances of the given type. All supported [mark types](Marks) are available. For example, `rect:mousedown` captures all `mousedown` events that occur on rect marks.|
| * @markName             | Filters for events that occur on marks with the given name. For example, `@cell:mousemove` captures all `mousemove` events that occur within the mark named `cell`.|
| * CSS selector          | The full gamut of CSS selectors can be used to capture events on elements that exist outside the visualization. D3 is used to capture these events, and the custom event functions, described above, are not available. For example `#header:mouseover` captures `mouseover` events that occur on the HTML element with ID `header`.|
| eventStream<b>[filterExpr]</b> | Filters for events that match the given expression. The filter expression is specified using normal JavaScript syntax, and the `event` and `datum` variables are also available. Multiple expressions can also be specified. For example, `mousedown[eventX() > 5][eventY() < 100]` captures `mousedown` events which occur at least 5px horizontally, and no more than 100px vertically within the visualization.| 
| streamA<b>,</b> streamB        | Merges individual event streams into a single stream with the constituent events interleaved correctly. For example, `@cell:mousemove, mousedown[eventX() > 5][eventY() < 100]` produces a single stream of `@cell:mousemove` and `mousedown[eventX() > 5][eventY() < 100]` events, interleaved as they occur.| 
| <b>[streamA, streamB] > </b> streamC  | Captures _streamC_ events that occur between _streamA_ and _streamB_. For example, `[mousedown, mouseup] > mousemove` describes a stream of `mousemove` events that occur between a `mousedown` and a `mouseup`, otherwise known as a "drag" stream.| 

### Scoped Scale References

When defining an interactive behaviour, it can often be useful to mix between the visual/pixel space of the rendered visualization, and its backing data space. For example, with brushing &amp; linking a scatterplot matrix, the pixel extents of the brush are inverted to produce a data range to highlight points across all cells of the matrix. Scoped scale references help with this. They can be either an object, or the name of a top-level scale. In the latter case, the scale transform is applied as normal (transforming the signal value from the data domain to the visual range). 

If the scope scale reference is an object, the following properties are available

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| name                | String              | The name of the scale.|
| scope               | String, [SignalRef](#using-and-referencing-signals)| An expression or signal reference to a group mark that contains this scale. If no scope is specified, the scale is expected to be defined at the top-level.|
| invert              | Boolean             | If true, an inverse of the scale transform is applied (i.e., transforming the signal value from the visual range to the data domain).|

For example, in the following specification, the group mark named `cell` is captured by the corresponding signal. This signal is used to lookup the `x` scale, which subsequently inverts the value of the `start_x` signal. 

```json
{
  "signals": [
    {
      "name": "cell",
      "streams": [
        {"type": "@cell:mousedown", "expr": "eventGroup('cell')"}
      ]
    },
    {
      "name": "start_x",
      "init": 0,
      "streams": [
        {
          "type": "mousedown", 
          "expr": "eventX()",
          "scale": {"name": "x", "scope": {"signal": "cell"}, "invert": true}
        }
      ]
    }
  ],

  "marks": [
    {
      "name": "cell",
      "type": "group",

      "scales": [{"name": "x", ...}],
      ...
    }
  ]
}
```

### Using and Referencing Signals

Once defined, signals can be used throughout a specification. They are available for use within all expressions (e.g., with the filter or formula transforms), and a `signal` property is available with mark property value references. For other areas of the specification, a special signal reference ("SignalRef") can be used. A SignalRef is an object with a single `signal` property. For example, 

```json
{
  "signals": [{"name": "xMin", ...}, {"name": "xMax", ...}],

  "scales": [
    {
      "name": "x",
      "range": "width",
      "domainMin": {"signal": "xMin"},
      "domainMax": {"signal": "xMax"}
    }
  ]
}
```
Dot notation can also be used to access nested signal values. For example,

```json
{
  "signals": [
    {
      "name": "brush_start",
      "init": {"x": 0, "y": 0},
      "streams": [{"type": "mousedown", "expr": "{x: eventX(), y: eventY()}"}]
    }
  ],

  "marks": [
    {
      "type": "rect",
      "properties": {
        "update": {
          "x": {"signal": "brush_start.x"},
          ...
        }
      }
    }
  ]
} 
```
