> [Wiki](Home) ▸ [[Documentation]] ▸ **Interaction Scenarios**

This scratchpad lists various scenarios that might be handled by adding support for declarative interaction to Vega as suggested at https://github.com/vega/vega/wiki/Contribute. Some of these interaction techniques are fairly standard and described at: http://en.wikipedia.org/wiki/Interactive_visual_analysis.

### Annotation

It is common to want to see the specific values of a data point when mousing over and/or clicking/tapping the corresponding mark. It should be possible to have the data displayed in one way when mousing over, and another way when clicking/tapping. For example, mousing over a data point might show a small tooltip near that data point with the X/Y values, but clicking on that data point might show the full data values in a fixed-position box on the canvas.

### Layout & Resizability

Reactively laid out documents have become increasingly important as the range of screen sizes grows wider. While simple resizing is currently possible in Vega by re-rendering the view in window.onresize, there are other features that would be useful:

1. Span-sensitive axes ticks: automatic tick generation should take into account the available display space to prevent tick labels from overlapping each other. Axes with quantitative scales could simply spread their ticks farther apart to prevent overlap, while axes with ordinal scales could choose to display the tick label only for every Nth label (e.g., the ordinal scale "North", "South", "East", "West" could just display "North" and "East" if there isn't enough room to display all four).
2. Occlusion prevention: when data points have text labels attached to them, the labels could opt to only be shown if they do not overlap any other labels. Optionally, a precedence scale might be set to allow labels with lower precedence to be hidden before labels with a higher precedence.
3. Proportional layout of groups: for specs that have multiple groups representing separate visualizations (e.g. a separate bar chart and geographic map in a single spec), you might want to be able to declaratively specify that the map should take up 80% of the width and the bar chart take 20% of the width. It might be worth considering using a declarative constraint specification and solver system like http://en.wikipedia.org/wiki/Cassowary_constraint_solver, although this could be overkill.

### Brushing/Selecting

It should be possible to select data points using common UI methods. Rubber banding could be used for bulk selection, and command-click could be used to add individual marks to the current selection.

We may also consider persisting the selection in the URL fragment identifier so that a link to a visualization with the current selection can be shared.

### Linking

Separate groups should be able to share a common selection, such that selecting the marks in one group will also select the corresponding marks in the other group. This is described at http://en.wikipedia.org/wiki/Brushing_and_linking and a good example can be seen at http://bl.ocks.org/mbostock/4063663.

### Animated paging

The existing "facet" transform could be used to create "temporal facets" that would apply each facet's data to a a single group mark over time. The result would be a chart that animates between subsets to the data. Interaction would require a widget that allows play/pause, step forward/backward, adjust speed, and a scrubber for manually scrolling to the correct position. A standard HTML control could be provided by Vega with an API to allow alternative implementations that call into the runtime.

### Zooming & Panning

Similar to resizability, but the viewport size would remain constant and the displayed data range would be adjusted. This could be linked directly to the scales that map to geometric bounds.

### Filtering & sorting

Dynamically allowing the user to filter and sort data could be useful.

### Data viewing & export

Viewing the raw tabular data corresponding to a selection of marks would be useful, as would be the ability to save it as CSV/TSV.

### Transformative Drilling

Some drilling might just act as a combination of filter and zoom: e.g., clicking on a segment of a treemap could zoom into that sector of the visualization. But another form of drilling could be where you click on a point on a map and the visualization turns into a bar chart describing a subset of the data. Or clicking on one bar of a stacked bar graph could turn it into a pie chart with that bar's data. The same technique used for brushing & linking might be re-used for transformative drilling.
