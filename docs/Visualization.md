## VISUALIZATION

A visualization is the top-level object in Vega, and is the container for all visual elements. A visualization consists of a rectangular canvas (the space in which the visual elements reside) and a viewport (a window on to that canvas). In most cases the two are the same size; if the viewport is smaller then the region should be scrollable.

Within the visualization is a sub-region called the "data rectangle". All marks reside within the data rectangle. By default the data rectangle fills up the full canvas. Optional padding adds space between the borders of the canvas and data rectangle into which axes can be placed. Note that the total width and height of a visualization is determined by the data rectangle size plus the padding values.

### Visualization Properties 

| Property      | Type          | Description    |
| :------------ |:-------------:| :------------- |
| name          | String        | A unique name for the visualization specification. |
| width         | Integer       | The total width, in pixels, of the data rectangle. Default is 500 if undefined. |
| height        | Integer       | The total height, in pixels, of the data rectangle. Default is 500 if undefined. |
| viewport      | Array&lt;Integer&gt;   | The width and height of the on-screen viewport, in pixels. If necessary, clipping and scrolling will be applied. |
| padding       | Number, Object, String | The internal padding, in pixels, from the edge of the visualization canvas to the data rectangle. If an object is provided, it must include {top, left, right, bottom} properties. Two string values are also acceptable: `"auto"` (the default) and `"strict"`. Auto-padding computes the padding dynamically based on the contents of the visualization. __All__ marks, including axes and legends, are used to compute the necessary padding. "Strict" auto-padding attempts to adjust the padding such that the overall width and height of the visualization is unchanged. This mode can cause the visualization's __width__ and __height__ parameters to be adjusted such that the total size, including padding, remains constant. Note that in some cases strict padding is not possible; for example, if the axis labels are much larger than the data rectangle. |

