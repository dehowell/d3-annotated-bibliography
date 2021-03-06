# D3: An Annotated Bibliography

- [Articles in the D3 queue](https://pinboard.in/u:dehowell/t:d3/unread/)

## References

### Mike Bostock: "Let's Make a Bar Chart"

([link][1])

The canonical D3 tutorial, introducing selections, the method-chaining style of the D3 API, using a data join to create elements, mapping data to properties of DOM nodes, and introducing scales to translate between the bounds of your data and the bounds of the elements in the page.

Coming back to this tutorial after having read many other sources, I'm struck by how much of D3's conceptual structure Bostock delivers here. Although, I know that some of it flew right over my head the first time I worked through this example:

> The data operator returns the update selection. The enter and exit selections hang off the update selection, so you can ignore them if you don’t need to add or remove elements.

I found getting used to which selections you need to savee references to an when one of the trickier bits of D3. But Bostock actually explains it pretty clearly:

> Since method chaining can only be used to descend into the document hierarchy, use `var` to keep references to selections and go back up.

It's just that it doesn't stick until you've been confused by a few examples -and start remembering which API methods return a new selection.

### Mike Bostock: "Let's Make a Bar Chart, II"

([link][2])

On to part two, which introduces basic SVG, loading data from external files, and data structured as an array of objects instead of just an array of values. Bostock also introduces the use of a `<g>` tag to wrap multiple visual elements that need to be positioned together. But this is a little bit of a subtle technique, because it involves positioning the group relative to the SVG and then positioning its children _relative to the `<g>` tag_.

In the previous "Let's Make a Bar Chart", Bostock didn't save a reference to the enter selection where he added the `<div>` bars. In the SVG example, he starts with the `<g>` and saves the selection of all these elements, positioned along the y-axis according to their order in the data set.

```javascript

var bar = chart.selectAll("g")
    .data(data)
  .enter().append("g")
    .attr("transform", function(d, i) {
        return "translate(0," + i * barHeight + ")";
    });
```

The `bar` variable now references the selection of `<g>` nodes. Each new chain off this selection can add a new child element to each `<g>`, inheriting the same datum as its parent.

```javascript
bar.append("rect")
    .attr("width", x)
    .attr("height", barHeight - 1);

bar.append("text")
    .attr("x", function(d) { return x(d) - 3; })
    .attr("y", barHeight / 2)
    .attr("dy", ".35em")
    .text(function(d) { return d; });
```

### Mike Bostock: "Let's Make a Bar Chart, III"

([link][3])

Part three of the bar chart tutorial introduces **ordinal scales**. In an ordinal scale, the input data set takes one of a discrete set of values. Unless we're mapping an ordinal variable to colors, the geometric parameters of SVG elements need to be numerical. Setting up an ordinal scale is more involved than a linear scale because there are simply more ways you might want to map a single value to a range of numbers.

You can either be completely explicit:

```javascript
var x = d3.scale.ordinal()
    .domain(["A", "B", "C", "D", "E", "F"])
    .range([0, 1, 2, 3, 4, 5]);
```

Or use `rangeBands` or `rangePoints` to generate the output values for you from a provided range. Bostock describes the differences between these:

> The rangeBands method computes range values so as to divide the chart area into evenly-spaced, evenly-sized bands, as in a bar chart. The similar rangePoints method computes evenly-spaced range values as in a scatterplot.

Each of these methods has a corresponding rounded version, which forces the values to be nearest integers for the sake of pixel-kindness. Using these to try and understand the difference between bands and points:

```javascript
var data = ["A", "B", "C", "D", "E", "F"],
    scale = d3.scale.ordinal() .domain(data);

_.map(data, scale.rangeRoundBands([0, 100])  // => [2, 18, 34, 50, 66, 82]
_.map(data, scale.rangeRoundPoints([0, 100]) // => [0, 20, 40, 60, 80, 100]
```


It really does come down to bands requiring some width, so the range is subdivided into evenly-spaced regions. Points are assumed to not have width, so the start and of the range can themselves be used for positioning data.

Part three also introduces axes, a convention for handling margins, and gives a hat tip to the `ggplot` library for R.

### Mike Bostock: "Margin Convention"

([link][4])

This article lays out a convention for specifying margins on a D3 graphic that lets you customize them per-side and then mostly ignore them in subsequent calculations - at least if you're working with SVG, where you can wrap the entire plot in a `<g>` and have that handle the margins for you:

```javascript
var svg = d3.select("body").append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
  .append("g")
    .attr("transform", "translate(" + margin.left + "," + margin.top + ")");
```

### Hadley Wickham: ggplot2 `geom_point` documentation 

([link][5])

Although `ggplot2` is higher level library (and, *I know*, for R and not the web), there's some conceptual similarity between it and D3 that I think can help illuminate the latter.

The anatomy of a simple scatter plot in ggplot, using R's built-in mtcars sample data set, demonstrates some of the major building blocks: 

```r
# Bind the mtcars dataset to the chart.
# 
# Map each element of the dataset to a point,
# using the wt variable for the x position
# and the mpg variable for the y position.

ggplot(mtcars) + geom_point(aes(x = wt, y = mpg))
```

There are also pluggable scales in ggplot, although this example is using implicit linear scales.


```javascript
svg.selectAll("circle")
    // Bind the mtcars dataset to the SVG.
    .data(mtcars)
    .enter()
        // Represent each observation with a point.
        .append("circle")
        .attr({
            // Use the wt variable for the x position.
            cx: function(d) { return x(d.wt); },
            // And the mpg variable for the y.
            cy: function(d) { return y(d.mpg); }
        });
```

ggplot is higher-level here primarily because it encapsulates the visual options in its `geom_*` function, each of which represents a different layer you can add on to the chart. D3 has no such nicety so that you work directly with DOM nodes and SVG nodes.

### Mike Bostock: "General Update Pattern, I"

([link][6])

Bostock explains the different types of selection, using just text elements for a barebones example. D3 binds data to DOM nodes by setting the datum as the `__data__` property of the node. Whenever you call `data(dataset)` on a D3 selection, the provided dataset is matched up with the DOM nodes in the selection. In the example, `text` is the _update selection_.

```javascript
var text = svg.selectAll("text").data(data);
```

A DOM node bound to a datum that is also present in the new dataset supplied here is part of this selection. Any elements in `data` that aren't already bound to the DOM go to `text.enter()`. Elements that were bound to a DOM node, but aren't in `data` are returned in the `text.exit()` selection.

### Mike Bostock: "General Update Pattern, II"

([link][7])

When updating a selection with new data, D3 uses a _key function_ provided with the data set to match up elements of the data set with the elements in the DOM to compute the update, enter, and exit selections.

I cloned the  [gist][7a] for this article and modified the text selection to key by index (the default behavior) to see how things break.

```diff
   var text = svg.selectAll("text")
-    .data(data, function(d) { return d; });
+    .data(data);
```

With this change, an element is included in the update selection only if the new dataset also has an element in that position. Letters of the alphabet are duplicated, along with other weird effects. It's worth [looking at][7b] with the console open to get a better feel.

[7a]: https://gist.github.com/mbostock/3808221
[7b]: http://bl.ocks.org/dehowell/e255bbd54897577b180e

### Mike Bostock: "General Update Pattern, III"

([link][8])

Transitions help make the role of the key function clear. Moral of the story: if you're going to animate your data, you probably need to use a key function.

### Mike Bostock: "Thinking With Joins"

([link][9])

Discusses the motivation behind the declarative style of D3 (as opposed to iterating manually over your data to create and modify DOM elements) as well as defining how the join between data and DOM elements translates to the enter, update, and exit selections.

The Venn diagram is handy:

- enter: The complement of the DOM elements with respect to the data.
- exit: The complement of the data with respect to the DOM elements.
- update:  The intersection of the data with the DOM elements.

### Mike Bostock: "Object Constancy"

([link][10])

> Animated transitions are pretty, but they also serve a purpose: they make it easier to follow the data.

Bostock touches briefly on key functions and transitions without much detail, but this article is more notable for the discussion of the _design_ reasons for animation.

### Mike Bostock: "Histogram"

([link][11])

D3 provides several "layout" objects, each of which restructures a data set to match the requirements of a particular presentation style. The first of these I've tried to wrap my head around is the histogram, which groups a flat data set into bins. Although the [documentation][11a] describes the resulting structure, I had to see it for myself in the console to wrap my head around what to do with the result.

I find adapting D3 examples to my own needs sometimes tricky and Bostock's own histogram example has a gotcha in it that I stumbled over. When he sets the width of the bars, he uses the x scale to translate it into pixels.

```javascript
bar.append("rect")
    .attr("x", 1)
    .attr("width", x(data[0].dx) - 1)
    .attr("height", function(d) { return height - y(d.y); });
```

That makes sense - the `dx` property on the bin is in units of the data set and needs to translated into pixels, right? When I tried this out on my own data (weight measurements collected from a wireless scale), the width came out as a negative number. A *really* negative number. The sample above contains a little shortcut - because the domain of the x scale starts at 0, it can be used to translate a difference in the data into a difference in pixels. But the domain of my weight data started well above zero, so any number less than the lower bound on my data (like the width of the bins) therefore comes out negative. My solution was to offset the width by the lower end of the domain on the scale first:

```javascript
bar.append("rect")
    .attr("x", 1)
    .attr("width", x(data[0].dx + x.domain()[0]) - 1)
    .attr("height", function(d) { return height - y(d.y); });
```

Yes, the x coordinate of the bar is a constant. That works here because Bostock wraps the bar and a text label in a `<g>` tag and then positions the group. Everything within can be relatively positioned.

```javascript
var bar = svg.selectAll(".bar")
    .data(data)
  .enter().append("g")
    .attr("class", "bar")
    .attr("transform", function(d) { return "translate(" + x(d.x) + "," + y(d.y) + ")"; });
```


[11a]: https://github.com/mbostock/d3/wiki/Histogram-Layout#_histogram "Histogram Layout"

### Mike Bostock: "Towards Reusable Charts"

([link][12])

Bostock describes a simple implementation of a reusable charting function, which itself has getter and setter methods for adjusting its configuration. The user of the charting function would invoke it something like:

```javascript
var chart = timeSeriesChart()
    .x(function(d) { return formatDate.parse(d.date); })
    .y(function(d) { return +d.price; });

var formatDate = d3.time.format("%b %Y");

// ...

d3.csv("sp500.csv", function(data) {
  d3.select("#example")
      .datum(data)
      .call(chart);
});
```

This has the merits of a simple API for the caller, but it still feels a little bit "Excel chart wizard" to me. The article itself ends by asking the question:

> We now have a strawman convention for reusable visualization components. Yet there is far more to cover as to what should be considered configuration or even a chart. Is a traditional chart typology the best choice?

My answer is "no". If I wanted traditional chart types, I wouldn't care about D3 in the first place.

### Ari Lerner and Victor Powell: _D3 on AngularJS_

([link][13])

A substantial chunk of _D3 on AngularJS_ is a review of just D3 on its own, but it's a pretty good survey for someone who has gotten past the basics already. The advice on integrating with AngularJS seems solid and mostly focuses on creating directives for wrapping the business of calling the D3 API, along with when to use `$apply` and `$watch` to keep D3 events integreted with updates in the AngularJS model.

---

David Howell, 2015


[1]: http://bost.ocks.org/mike/bar/ "Let's Make a Bar Chart"
[2]: http://bost.ocks.org/mike/bar/2/ "Let's Make a Bar Chart, II"
[3]: http://bost.ocks.org/mike/bar/3/ "Let's Make a Bar Chart, III"
[4]: http://bl.ocks.org/mbostock/3019563 "Margin Convention"
[5]: http://docs.ggplot2.org/current/geom_point.html "ggplot2: geom_point documentation"
[6]: http://bl.ocks.org/mbostock/3808218 "General Update Pattern, I"
[7]: http://bl.ocks.org/mbostock/3808221 "General Update Pattern, II"
[8]: http://bl.ocks.org/mbostock/3808234 "General Update Pattern, III"
[9]: http://bost.ocks.org/mike/join/ "Thinking With Joins"
[10]: http://bost.ocks.org/mike/constancy/ "Object Constancy"
[11]: http://bl.ocks.org/mbostock/3048450 "Histogram"
[12]: http://bost.ocks.org/mike/chart/ "Towards Reusable Charts"
[13]: https://leanpub.com/d3angularjs "D3 on AngularJS"