> [Wiki](Home) ▸ **Documentation**

__Vega__ is a _visualization grammar_: a declarative format for describing and creating data visualizations. Other, existing grammars ([Wilkinson's Grammar of Graphics](http://books.google.com/books/about/The_Grammar_of_Graphics.html?id=_kRX4LoFfGQC), [Wickham's ggplot2](http://ggplot2.org/) and [Tableau's VizQL](http://www.tableausoftware.com/products/technology)) focus on rapid generation of charts to support analysis. However, in service of rapid specification these systems make a number of decisions on behalf of the user, and also impose limits on the type of visualizations one can create. Vega is intended to be lower-level, enabling fine-gained control of the visualization design. Vega's model of visualization design is intimately influenced by the [Protovis](http://protovis.org) and [D3](http://d3js.org) frameworks, and [research](http://idl.cs.washington.edu) at Stanford and the University of Washington.

A Vega specification is simply a JSON object that describes an interactive visualization. A specification describes the data sets used, scale transforms and encoding algorithms, axes and legends, visual marks (rectangles, lines, shapes, etc) whose properties may depend on the data, and signals and predicates that modify the visualization in response to user interaction. In effect, Vega specifications can serve as a "file format" for custom visualization designs. These specifications may be read and interpreted by a runtime system to dynamically create visualizations, or a specification may be cross-compiled to provide a reusable visualization component, in the form of editable code for a specific visualization framework (such as D3).

While intended to be a general (platform-agnostic) specification, Vega has been primary designed for creating web-based visualizations using HTML5 technologies such as Canvas and SVG (Scalable Vector Graphics). This focus has inevitably sculpted the specification language to the web domain.

## Overview

A Vega __specification__ is simply a [JSON](http://en.wikipedia.org/wiki/JSON) object that describes an interactive visualization design. We will refer to the top-level object as the _Visualization_ specification. A [Visualization](Visualization) consists of basic properties (such as the width and height of the view) and definitions for: the [Data](Data) to visualize, [Scales](Scales) that map data values to visual values, [Axes](Axes) and [Legends](Legends) that visualize these scales, graphical [Marks](Marks) (such as rectangles, circles, arcs, etc) that visualize the data, [Signals](Signals) to capture user interaction and [Predicates](Predicates) to modify the visualization in response.

To start learning Vega, we recommend first working through the introductory __[Bar Chart Tutorial](Tutorial)__.

Next, you can explore the online examples in the __[Vega Editor](http://vega.github.io/vega-editor/)__ and read the more detailed documentation below.

## Specification Components

Documentation for each major component of a Vega specification can be found on these wiki pages:

| Page          | Description  |
| :------------ | :------------|
| [Visualization](Visualization) | Top-level visualization properties. |
| [Data](Data)                   | Define and load data to visualize.  |
| [Data Transforms](Data-Transforms) | Transform data prior to visualization. |
| [Scales](Scales)               | Map data properties to visual properties using scales. |
| [Axes](Axes)                   | Axes visualize scales for spatial encodings. |
| [Legends](Legends)             | Legends visualize scales for color, shape and size encodings. |
| [Marks](Marks)                 | Visualize data using various graphical marks. |
| [Group Marks](Group-Marks)     | A special kind of mark that can contain other marks. |
| [Signals](Signals)             | Dynamic variables to drive interactive behaviors. |
| [Predicates](Predicates)       | Conditions (or "selections") to modify mark properties. |
| [Runtime](Runtime)             | Deploying and using the browser-based Vega runtime. |
| [Headless Mode](Headless-Mode) | Server-side Vega using "headless" rendering. |
