> [Wiki](Home) ▸ [[Documentation]] ▸ **Runtime**

The Vega runtime takes a Vega spec as input, parses it, and generates an interactive visualization in your browser using HTML5 technologies.

### Basic Use

First, you'll need to create an HTML document to house your Vega visualization. Here's a scaffolding:
```html
<html>
  <head>
    <title>Vega Scaffold</title>
    <script src="http://vega.github.io/vega-editor/vendor/d3.min.js"></script>
    <script src="http://vega.github.io/vega-editor/vendor/d3.geo.projection.min.js"></script>
    <script src="http://vega.github.io/vega-editor/vendor/topojson.js"></script>
    <script src="http://vega.github.io/vega/vega.min.js"></script>
  </head>
  <body>
    <div id="vis"></div>
  </body>
<script type="text/javascript">
// parse a spec and create a visualization view
function parse(spec) {
  vg.parse.spec(spec, function(chart) { chart({el:"#vis"}).update(); });
}
parse("uri/to/your/vega/spec.json");
</script>
</html>
```

The HTML headers import the necessary libraries: D3 and Vega. Note that the `d3.geo.projection.min.js` and `topojson.js` imports are _optional_, and only needed if you wish to use [extended geographic projections](https://github.com/d3/d3-geo-projection/) or [TopoJSON](https://github.com/mbostock/topojson/wiki)-formatted data.

The main starting point is then the method `vg.parse.spec`, which takes either a Vega JSON specification or a URL as input. The specification file is then loaded and parsed into a chart component.

**Advanced Use**

An optional _config_ object can be passed as the third parameter to `vg.parse.spec` in order to override the [default configuration options](https://github.com/vega/vega/blob/master/src/core/config.js) for the current specification.

Assuming no errors occur, the compiled chart component is returned by a callback. The callback argument `chart` above is a function that serves as a constructor for new chart view instances. This constructor takes an options hash as a single argument. The following options are supported:

| Option              | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| el                  | DOMElement, String  | The DOM element in which the chart should reside. The input value can either be a DOM object or a CSS selector. Note that Vega may remove or change any existing content in the DOM element. If no element is specified, Vega will render in [Headless Mode](Headless).|
| data                | Object  (optional)  | An object containing named data sets. The data argument can be used to bind data sets at runtime for dynamic, reusable chart components. Data sets whose names do not match any data definitions in the Vega specification will be ignored.|
| hover               | Boolean (optional)  | Determines if the chart should use the default hover behavior, which is to invoke the `"hover"` property set upon mouseover and the `"update"` property set upon mouseout. This option defaults to true if undefined. If set to `false`, the returned chart view instance will have no registered event listeners.|
| renderer            | String  (optional)  | Specifies what rendering system to use. A value of `"canvas"` (the default) tells Vega to use the HTML5 canvas tag to render visualizations. A value of `"svg"` causes Scalable Vector Graphics (SVG) to be used instead.|

Once a visualization component has been instantiated and configured, the `update()` method initiates visual encoding and rendering.

### View Component API

The `chart` constructor returns an instance of `vg.View`, which provides a public API for further runtime customization of the visualization, including the addition of event listeners for interaction. The documentation below describes the available methods of the view component. Each can be invoked using a method chaining style. For example, the following code changes the size and rendering method for an existing view: `view.width(500).height(200).renderer("svg").update()`.

| Property            | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| width               | Integer             | Sets the width of the view canvas, in pixels.|
| height              | Integer             | Sets the height of the view canvas, in pixels.|
| padding             | Object              | Sets the padding of the view component. The input object _must_ include `left`, `right`, `top`, and `bottom` properties, each with an integer value.|
| viewport            | Array&lt;Integer&gt;| Sets the visible viewport size. The input should be a two-element array with integer values indicating the width and height of the viewport, in pixels. If the viewport size is smaller than the __width__ or __height__ properties, the view component will include scroll bars.|
| renderer            | String              | Sets which renderer to use to draw the Vega visualization. The legal values are `"canvas"` for HTML5 2D canvas drawing (the default) and `"svg"` for Scalable Vector Graphics.|
| data                | String, Object      | Updates the current data bindings. For more information, see [Streaming Data](Streaming-Data).|
| signal              | String              | A getter/setter for signal values. The first argument is a signal name. If no second argument is specified, returns the signal's current value. If a second argument is specified, sets the signal value. The _update_ method must be called to reflect new signal values on the visualization. |
| initialize          | String, DOMElement  | Initializes the view component for use under the input DOM element. The input value can either be a DOM object or a CSS selector. Note that Vega may remove or change any existing content in the DOM element. Clients should rarely if ever invoke this method, as it is invoked by the chart view constructor.|
| render              | Array&lt;SceneItems&gt;| Causes the current scene to be redrawn as-is. If no arguments are provided, the full scene is redrawn. If an array of scenegraph items are provided as input, just those items will be redrawn.|
| update              | Object              | Invokes an update for the visualization. If no arguments are provided, the full scenegraph will be constructed according to the current data sets, all visual encoders will be run, and then the scene will be rendered. The update function takes an optional options hash as its single argument. The following options are supported:
| _props_             | String              | A string indicating the name of the mark property set to run (e.g., `"update"` or `"hover"`). This allows targeted invocation of specific mark property encodings. This option should only be used _after_ a full build of the visualization has been performed.|
| _items_             | SceneItem, Array&lt;SceneItem&gt;| A single scenegraph item or an array of scenegraph items to update. All visual encoding and rendering will be limited to just these input items. This option should only be used _after_ a full build of the visualization has been performed.|
| _duration_          | Number              | The length of an animated transition for this update, in milliseconds. If unspecified, a static (zero second) transition will be used.| 
| _ease_              | String              | The easing function for the animated transition. The supported easing types are `linear`, `quad`, `cubic`, `sin`, `exp`, `circle`, and `bounce`, plus the modifiers `in`, `out`, `in-out`, and `out-in`. The default is `cubic-in-out`. For more details please see the [D3 ease function documentation](https://github.com/mbostock/d3/wiki/Transitions#wiki-d3_ease).|

##### Examples

Update the visualization view. Invokes the "update" property set for all marks.
```
view.update()
```

Update the view by invoking the "hover" property set for the given scenegraph item.
```
view.update({props:"hover", items:item})
```

Update the visualization view using a 500 millisecond (0.5s) animated transition.
```
view.update({duration: 500})
```

Update the visualization view using a 500 millisecond (0.5s) animated transition and a "bounce-in" easing function.
```
view.update({duration:500, ease:"bounce-in"})
```

Updates the view to show the hover properties of the given scenegraph item, using a 500ms animated transition.
```
view.update({props:"hover", items:item, duration:500, ease:"bounce-in"})
```

| Name                | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| on                  | String, Function    | Adds an event listener. The first argument should be the event name to listen to (event names of the form `"click.custom"` are allowed to enable easier removal of specific listeners; the actual DOM event name is assumed to be the text before the first dot character). The second argument should be a callback function that accepts two arguments: a DOM Event object and a scenegraph item.|

##### Examples

Adds a listener function to receive "mouseover" events. This particular listener simply prints the scenegraph item being hovered over.
```
view.on("mouseover", function(event, item) { console.log(item); })
```

Adds a listener function to receive "click" events. This particular listener invokes a custom "click" property set for the selected item; the "click" properties must be defined within the mark specification.
```
view.on("click", function(event, item) { view.update("click", item); })
```

Adds the event listener function `func` to receive "mouseover" events, using a special suffix to aid targeted removal of the listener later on.
```
view.on("mouseover.custom", func)
```

| Name                | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| off                 | String, Function    | Removes one or more event listeners. The first argument indicates the event name for which to remove listeners. If only one argument is provided, all listeners registered with that same string will be removed. The second (optional) argument should be a specific event listener callback to remove.|

##### Examples

Removes all event listeners registered for the "mouseover" event. Listeners registered with a special suffix (e.g., "mouseover.custom") will not be removed.
```
view.off("mouseover")
```

Removes all event listeners registered for the "mouseover.custom" event. Listeners registered with no suffix ("mouseover") or a different suffix ("mouseover.mine") will not be removed.
```
view.off("mouseover.custom")
```

Removes the specific event listener function `func` registered on "mouseout" events.
```
view.off("mouseout", func)
```

| Name                | Type                | Description  |
| :------------------ |:-------------------:| :------------|
| onSignal            | String, Function    | Adds a signal listener. The first argument should be the name of the signal to listen to. The second argument should be a callback function that accepts two arguments: the name of the signal, and its current value.| 
| offSignal           | String, Function    | Removes one or more signal listeners. The first argument should be the name of the signal for which to remove listeners. If only one argument is provided, all listeners registered on the specified signal will be removed. The second (optional) argument should be a specific signal listener callback to remove.|


