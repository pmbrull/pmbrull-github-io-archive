Udacity

Designers, programmers and story tellers (communication).

It is mandatory being able to communicate the results of any study and keeping in mind what use there is for that. If there is no way to extract real value, a 99% accuracy model is mainly useless. You need to focus on who is going to use that analysis and for what, and always aim to provide excellence in what that customer needs, not what you think it needs.

Source Ben Fry: Computer Science (acquire - parse), Exploratory Data Analysis (EDA) and Modelling (filter- mine) - Graphic Design (represent, refine) - Infovis (information visualiaztion) and HCI human computer interaction (interact). With data visualization we are not just giving our customer the cake baked by the ML models, but bringing a key point in the whole *Data Science* process.

This process is not linear. Once we visualize the data, we may need to go back in the acquiring part as something is not making real sense, or in the other way around, we may want to stress some key element discovered in early stages in the final design.

Aim: enable the user to discover insights for themselves.

Create engaging graphics,

Dash vs Shiny (reactivity made easier vs cached values as jsonized children)

The bandwith of our senses

Importance of data visualization: Anscombe's quartet. Otherwise, you are surely missing patterns and deviations

D3: data driven documents (webpage source - HTML). Binds data to the document. DOM (document object model), generated during page load as it receives html source. We can access the DOM with javascript API,

```javascript
var script = document.createElement('script');
script.type = 'text/javascript';
script.src = "https://d3js.org/d3.v5.min.js"
```

document.querySelector() returns a DOM node, while d3.select(), an array with D3 objects, thus we will be able to keep chaining D3 methods, as most D3 methods return a selection of the element they were called on.

```javascript
// Select from parent to child elements: Nested Selections
d3.select('#header-logo img').attr('alt', 'logo')
```

We are saying: go to element with ID *header-logo*, and from there, select the *img* node.

```javascript
svg = d3.select('.main').append('svg').attr({width: 600, height: 300})
```

> OBS: For SVG, y coordinate gets higher going down. The opposite as in Maths.

`.data(d).enter()` is a subselect of the data d that has not yet been used in the code. ie:

```javascript
chart.selectAll('g').data(d).append('g')
```

Add a group for each data value d not yet used.

Digest a graphic. Easy to digest.

CHart types = visual encodings + data types + relationship

Extracted from the course: Basic chart choices

**C** *bar chart* - highlights individual values, supports comparisons, and can show rankings or deviations

**A** *boxplot* - shows distributions and quantiles, especially useful when comparing distributions

**B** *pie chart* - shows part-to-whole relationship and best suited for one category; poor for making comparisons 

**D** *stacked bar chart* - shows part-to-whole relationship and best suited for showing composition within categories and totals

**D** *bubble chart* - shows how three or more sets of values vary; shows correlation

**B** *line chart* - shows overall changes and patterns, usually over equally spaced intervals of time

**A** *map* - values are encoded on physical locations and  patterns may be drawn by comparing locations

**C** *scatterplot* - shows how two pair sets of values (for example height and shoe size) vary; shows correlation



Choropleth = geographic + color; cartogram = geographic + size; dot map = geographic + shape

Make use of pre-attentive process!! 200-250 miliseconds to ingest the information required: i.e: color (same amount of time as reading facial expressions). Pre-attentive attributes: color (hue or intensity), form, spacial position or movement.

Do you need color? Only use it to highlight some special category, for example, in a bar chart. Avoid primary colors. Use medium hues or pastels.

ChartJunk: distracts the audience from the important part of the visualization or adds no useful information.

the data-ink ratio is defined as the ratio of the amount of ink used to describe the data to the amount of ink in the graphic. The higher the better.

![https://d17h27t6h515a5.cloudfront.net/topher/2016/September/57df5e83_iris-box-plot-minimal/iris-box-plot-minimal.png](https://d17h27t6h515a5.cloudfront.net/topher/2016/September/57df5e83_iris-box-plot-minimal/iris-box-plot-minimal.png)

### Don't go overboard

Just a reminder - we don't always need to strive for minimal  encodings. Adding various design elements can be useful for clarity and  to emphasize the story we want to tell.

## Lie Factor

lie factor = size of the effect shown in the graphic / size of the effect shown in the data. We want it in [0.95, 1.05], ideally, 1. Otherwise, it is not accurate.![lie-factor](/home/pmbrull/Documents/D3/lie-factor.bmp)

## Grammar of Graphics

The Grammar of Graphics is a visualization theory developed by [Leland Wilkinson](http://en.wikipedia.org/wiki/Leland_Wilkinson) in 1999 with the publication of the eponymous [book](http://www.springer.com/statistics/computational+statistics/book/978-0-387-24544-7).

It is quite an extensive theory which has influenced the development  of graphics and visualization libraries alike (including D3 and its  precursors), but in this class you will focus on 3 of its key  principles:

1. Separation of data from aesthetics
2. Definition of common plot/chart elements
3. Composition of these common elements

### Separation of Concerns

You just saw some of the benefits of separating the data from the  visual presentation of that data in the previous videos. The main  take-aways are:

* Independently transform data and present data
* Delegate work and responsibilities
  * Engineer focuses on data manipulation
  * Designer focuses on visual encoding of data
* Present multiple visual representations of a dataset
  * Ex: Bubble chart and line chart show [different](http://dc-js.github.io/dc.js/) facets of a dataset.

### Common Elements

When thinking about creating a chart or graphic, it is often helpful  to visually decompose what you want to achieve.  In previous videos you  saw how to abstract a chart into more basic visual encodings.  In the  map example, you saw that a choropleth is a combination of geography and  color while a cartogram is a combination of geography and size.  When  talking about **composable** elements, a few of the most common are:

- Coordinate System (cartesian vs. radial/polar)
- Scales (linear, logarithmic, etc.)
- Text annotations
- Shape (lines, circles, etc.)
- Data Types (Categorical, Continuous, etc.)

### Composition

The beauty of the Grammar of Graphics surfaces when you combine these  common components.  For example, you can create a bar chart by mapping a  value in the data to the height of the bar in cartesian space, but you  can also can also map these values in polar coordinates, in which the  data value corresponds to the radial degree of a slice, to get a pie  chart.

- Categorical + Continuous x Cartesian = Bar Chart
- Categorical + Continuous x Polar = Pie Chart
- Continuous + Continuous x Cartesian = Scatter Chart

And you can create a plethora of other charts by combining these common  components in different ways.  How might you achieve a line plot with a  logarithmic scale from these common components?



Separately think about visual elements and structure of data allows us to transform data without changing the visual representation and allows for collaboration across teams.



![grammar-of-graphics-pipeline](/home/pmbrull/Documents/D3/grammar-of-graphics-pipeline.bmp)

**E** [d3.selection.append](https://github.com/mbostock/d3/wiki/Selections#append)
inserts HTML or SVG elements into a web page

**C** [d3.selection.attr](https://github.com/mbostock/d3/wiki/Selections#attr)
changes a characteristic of an element such as position or fill

**D** [d3.json](https://github.com/mbostock/d3/wiki/Requests#d3_json)
loads a data file and returns an array of Javascript objects

**A** [d3.layout ](https://github.com/mbostock/d3/wiki/API-Reference#d3layout-layouts)
applies common transformations on predefined chart objects

**B** [d3.nest](https://github.com/mbostock/d3/wiki/Arrays#d3_nest)
groups data based on particular keys and returns an array of JSON

**F** [d3.scale](https://github.com/mbostock/d3/wiki/API-Reference#d3scale-scales)
converts data to a pixel or color value that can be displayed



# Narrative Structures

types of bias:

### Author Bias

Cole and Matt's data visualizations contain author bias. That is, the  designers and presenters of the visualizations (knowing or unknowingly)  misrepresented data through visual encodings or other design choices  such as the chart type. If you'd like to use a 3D Pie Chart in the  future, please remember Andy Kriebel's statement "Friends Don't Let  Friends Use Pie Charts".

As the designer or presenter of data visualizations, your design  choices should establish trust between the reader and the graphic. Your  design choices should facilitate communication. Otherwise as Cole  mentioned, you risk the overall credibility of your message among  readers.

### Data Bias

Data bias arises from the process of collecting data. Systematic  measurement errors or faulty devices can bias raw data values, and  selection bias can lead to subgroups that are not representative of the  population of interest for a given question. Data bias and sampling  methods are beyond the scope of this class; however, we encourage you to learn more about these topics. Try reading articles about [data collection](http://en.wikipedia.org/wiki/Data_collection), [sampling methods](http://en.wikipedia.org/wiki/Sampling_(statistics)#Sampling_methods), or [other biases](http://en.wikipedia.org/wiki/Bias_(statistics)). Scott Murray touches upon one example of measurement errors in the next video.

## Reader Bias

The final type of bias that we'll cover is reader bias. Reader bias  encompasses any preconceived notions or assumptions that a reader brings  to interpreting a visualization.

The assumptions may pertain to a reader's domain knowledge or the  topic of the data visualization. For example if you know a little bit  about the World Cup, you are more likely to be aware of the tournament's  occurrence and the structures of the World Cup's competition and  stages.

Other assumptions may involve political, religious, or cultural  beliefs or a reader's familiarity with a specific chart type. You should  always consider your audience's background and familiarity with  graphics when designing a data visualization.

Both the designer (encoder) and the reader (decoder) ought to be  aware of bias. Communication can go awry due to the designer's choices  or the reader's interpretation of a graphic. This blurs the line of  misleading and lying in data visualization, and it is with analytical  thinking and due diligence that both designer and reader can partake in a  trusting exchange of information.

For more examples of author and reader bias, please read [Disinformation Visualization](https://visualisingadvocacy.org/blog/disinformation-visualization-how-lie-datavis) by Mushon Zer-Aviv.

## Types of narrative

![narrative](/home/pmbrull/Documents/D3/narrative.png)



![martini-glass](/home/pmbrull/Documents/D3/martini-glass.png)



## Joins in D3 Answers

**E** d3.select("svg")  - selects a container SVG element on the page

**A** .selectAll("circle") - creates an empty selection to bind data

**C** .data(data) - binds data to the empty selection

**D** .enter() - selects all bound data elements not displayed

**B** .append("circle") - creates SVG elements for the bound data

`.enter()` returns all elements of a selection for each row of data.tsv that aren't in index.html (on the page) `.exit()` returns all elements of a selection in index.html (on the page) that aren't bound to data

You can take a subset of an array in Javascript by using the `slice()` method.

[ slice()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice) documentation

The code `data.slice(0, 10);` would return the first 10  objects in the data array. Remember that indexing begins with 0. The  slice method excludes the last object (in this case, the object with  index 10).

`console.log(data.slice(0, 10));`
The code above would print the first 10 rows of the data file to the console in a sortable table.

[#](https://github.com/d3/d3-array/blob/master/README.md#extent) d3.**extent**(*array*[, *accessor*]) [<>](https://github.com/d3/d3-array/blob/master/src/extent.js)

Returns the [minimum](https://github.com/d3/d3-array/blob/master/README.md#min) and [maximum](https://github.com/d3/d3-array/blob/master/README.md#max) value in the given *array* using natural order. If the array is empty, returns [undefined, undefined]. An optional *accessor* function may be specified, which is equivalent to calling *array*.map(*accessor*) before computing the extent.

* Unary operator **+**: d.attendance = +d.attendance: tries to cast to numeric format.







