> [Wiki](Home) ▸ [[Documentation]] ▸ **Expressions**

To enable custom calculations, Vega includes its own _expression language_ for writing basic formulas. These expressions are used by the [filter](Data-Transforms#-filter) and [formula](wiki/Data-Transforms#-formula) transforms to modify data, and within [signal](Signals) definitions to process user input.

The expression language is a restricted subset of JavaScript. All basic arithmetic, logical and property access expressions are supported, as are boolean, number, string, object (`{}`) and array (`[]`) literals. 

To keep the expression language simple, secure and free of unwanted side effects, the following elements are disallowed: assignment operators (`=`, `+=` etc), pre/postfix updates (`++`), `new` expressions, and control flow statements (`for`, `while`, `switch`, etc). In addition, function calls involving nested properties (`foo.bar()`) are not allowed. Instead, the expression language supports a collection of functions defined in the top-level scope.

This wiki page documents the user-facing aspects of the expression language exposed by Vega. For those interested in implementation aspects, the bulk of the expression language &ndash; including parsing, code generation, and most of the constant and function definitions &ndash; are maintained in the [vega-expression module](http://github.com/vega/vega-expression).

### Bound Variables

__datum__

In all cases, Vega includes a `datum` variable in the top-level scope, corresponding to an input data object. To lookup object properties, use normal JavaScript syntax (`datum.value`).

__event__

If the expression is being invoked in response to an event (e.g., within a signal stream handler), an `event` variable is also included. This variable consists of a standard JavaScript DOM event, providing access to bound properties of the event, such as `event.metaKey` or `event.keyCode`.

__signals__

In addition, you can reference any signal value by name. For example, if you have defined a signal named `hover` within your Vega specification, you can refer to it directly within an expression (e.g., `hover.value`).

### Constants

In addition to the bound variables, the expression language includes the following constant values.

| Value         | Description    |
| :------------ | :------------- |
| `NaN`         | not a number (same as JavaScript literal `NaN`) |
| `E`           | the transcendental number _e_ (alias to `Math.E`) |
| `LN2`         | the natural log of 2 (alias to `Math.LN2`) |
| `LN10`        | the natural log of 10 (alias to `Math.LN10`) |
| `LOG2E`       | the base 2 logarithm of _e_ (alias to `Math.LOG2E`) |
| `LOG10E`      | the base 10 logarithm _e_ (alias to `Math.LOG10E`) |
| `PI`          | the transcendental number _&pi;_ (alias to `Math.PI`) |
| `SQRT1_2`     | the square root of 0.5 (alias to `Math.SQRT1_2`) |
| `SQRT2`       | the square root of 2 (alias to `Math.SQRT1_2`) |

### Functions

All expression language functions, grouped by category.

### Math Functions

| Function      | Description    |
| :------------ | :------------- |
| `isNaN`		| checks if a value is not-a-number (same as JavaScript `isNaN`) |
| `isFinite` 	| checks if a value is a finite number (same as JavaScript `isFinite`) |
| `abs` 		| absolute value (alias to `Math.abs`) |
| `acos` 		| trigonometric arccosine (alias to `Math.acos`) |
| `asin` 		| trigonometric arcsine (alias to `Math.asin`) |
| `atan` 		| trigonometric arctangent (alias to `Math.atan`) |
| `atan2` 		| returns the arctangent of the quotient of its arguments (alias to `Math.atan2`) |
| `ceil` 		| rounds to the nearest integer of greater value (alias to `Math.ceil`) |
| `cos` 		| trigonometric cosine (alias to `Math.cos`) |
| `exp` 		| raises _e_ to the provided exponent (alias to `Math.exp`) | 
| `floor` 		| rounds to the nearest integer of lower value (alias to `Math.floor`) |
| `log` 		| natural logarithm function (alias to `Math.log`) |
| `max`     	| return the maximum argument (alias to `Math.max`) |
| `min` 		| returns the minimum argument value (alias to `Math.min`) |
| `pow` 		| exponentiates the first argument by the second argument (alias to `Math.pow`) |
| `random` 		| generates a pseudo-random number in the range [0,1) (alias to `Math.random`) |
| `round` 		| rounds to the nearest integer (alias to `Math.round`) |
| `sin` 		| trigonometric sine (alias to `Math.sin`) |
| `sqrt` 		| square root function (alias to `Math.sqrt`) |
| `tan` 		| trigonometric tangent (alias to `Math.tan`) |
| `clamp` 		| restricts a value between a specified min and max (e.g. `clamp(value, min, max)`) |

### Date/Time Functions

| Function      | Description    |
| :------------ | :------------- |
| `now`			| returns the timestamp for the current time |
| `datetime`    | constructs a new `Date` instance |
| `date`        | returns the day of the month for a given date input, in local time |
| `day`  		| return the day of the week for a given date input, in local time |
| `year`        | returns the year for a given date input, in local time |
| `month`       | returns the (zero-based) month for a given date input, in local time |
| `hours`       | returns the hours component for a given date input, in local time |
| `minutes`     | returns the minutes component for a given date input, in local time |
| `seconds`     | returns the seconds component for a given date input, in local time |
| `milliseconds`| returns the millseconds component for a given date input, in local time |
| `time`        | returns the epoch-based timestamp a given date input |
| `timezoneoffset`  | returns the timezone offset from the local timezone to UTC for a given date input |
| `utcdate`     | returns the day of the month for a given date input, in UTC time |
| `utcday`      | returns the day of the week for a given date input, in UTC time |
| `utcyear`     | returns the year for a given date input, in UTC time |
| `utcmonth`    | returns the (zero-based) month for a given date input, in UTC time |
| `utchours`    | returns the hours component for a given date input, in UTC time |
| `utcminutes`  | returns the hours component for a given date input, in UTC time |
| `utcseconds`  | returns the hours component for a given date input, in UTC time | 
| `utcmilliseconds` | returns the hours component for a given date input, in UTC time |

### Sequence Functions

| Function      | Description    |
| :------------ | :------------- |
| `length` 		| returns the length of an array or string |
| `indexof` 	| returns the first index of an element (for array inputs) or substring (for string inputs) |
| `lastindexof` | returns the last index of an element (for array inputs) or substring (for string inputs) |

### String Functions

| Function      | Description    |
| :------------ | :------------- |
| `parseFloat` 	| parses a string to a floating-point value (same as JavaScript `parseFloat`) |
| `parseInt` 	| parses a string to an integer value (same as JavaScript `parseInt`) |
| `upper` 		| transforms a string to upper-case |
| `lower` 		| transforms a string to lower-case |
| `slice` 		| slices a string into a substring (alias to `String.slice`) |
| `substring` 	| extracts a substring from a string (alias to `String.substring`) |

### Regular Expression Functions

| Function      | Description    |
| :------------ | :------------- |
| `test` 		| evaluates a regular expression against a string, returning true if the string matches the pattern, false otherwise. _Example_: `test(/\\d{3}/, "32-21-9483") -> true` |

### Control Flow Functions

| Function      | Description    |
| :------------ | :------------- |
| `if`			| if the first argument evaluates true then the second argument is returned, otherwise the third argument is returned (`if(a, b, c)` is equivalent to `a ? b : c`) |

### Hyperlink Functions

| Function      | Description    |
| :------------ | :------------- |
| `open` 		| opens a hyperlink (alias to `window.open`). This function is only valid when running in the browser. It should not be invoked within a server-side (e.g., node.js) environment. |

### Event Functions

These functions are only legal when performing event processing, for example within a signal stream handler. Invoking these functions within an expression for a [filter](https://github.com/vega/vega/wiki/Data-Transforms#-filter) and [formula](https://github.com/vega/vega/wiki/Data-Transforms#-formula) transform will result in an error.

| Function      | Description    |
| :------------ | :------------- |
| `eventItem` 	| a zero-argument function that returns the current scenegraph item that is the subject of the event. |
| `eventGroup` 	| returns the scenegraph group mark item within which the current event has occurred. If no arguments are provided, the immediate parent group is returned. If a group name is provided, the matching ancestor group item is returned. |
| `eventX` 		| returns the x-coordinate for the current event. If no arguments are provided, the top-level coordinate space of the visualization is used. If a group name is provided, the coordinate-space of the matching ancestor group item is used. |
| `eventY` 		| returns the y-coordinate for the current event. If no arguments are provided, the top-level coordinate space of the visualization is used. If a group name is provided, the coordinate-space of the matching ancestor group item is used. |