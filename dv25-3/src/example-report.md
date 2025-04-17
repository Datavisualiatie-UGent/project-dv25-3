---
title: Social views of technology
toc: false
---

```js
const gender1 = await FileAttachment("data/gender1.csv").csv({ typed: true });
const gender2 = await FileAttachment("data/gender2.csv").csv({ typed: true });
const gender3 = await FileAttachment("data/gender3.csv").csv({ typed: true });
```


```js
function slope_graph(gender, {width, height} = {}){
  const occlusionY = ({radius = 6.5, ...options} = {}) => Plot.initializer(options, (data, facets, { y: {value: Y}, text: {value: T} }, {y: sy}, dimensions, context) => {
  for (const index of facets) {
    const unique = new Set();
    const nodes = Array.from(index, (i) => ({
      fx: 0,
      y: sy(Y[i]),
      visible: unique.has(T[i]) // remove duplicate labels
        ? false
        : !!unique.add(T[i]),
      i
    }));
    d3.forceSimulation(nodes.filter((d) => d.visible))
      .force("y", d3.forceY(({y}) => y)) // gravitate towards the original y
      .force("collide", d3.forceCollide().radius(radius)) // collide
      .stop()
      .tick(20);
    for (const { y, node, i, visible } of nodes) Y[i] = !visible ? NaN : y;
  }
  return {data, facets, channels: {y: {value: Y}}};
  });

  const countries = [...new Set(gender.map(d => d.country))];
  const years = [...new Set(gender.map(d => d.year))];
  const firstYear = Math.min(...years);
  const lastYear = Math.max(...years);

  return Plot.plot({
  height: 300,
  width: 320,
  style: { fontSize: "14px" },
  color: {
      type: "categorical",
      domain: countries
    },
  x: {axis: "top", type: "ordinal", tickFormat: "", inset: 90, label: null},
  y: {axis: null, inset: 20},
  marks: [
    Plot.ruleX([firstYear, lastYear], {stroke: "black", strokeDasharray: "4 2"}),
    Plot.line(gender, {x: "year", y: "percent", z: "country", stroke: "country"}),
    d3.groups(gender, (d) => d.year === 2021)
      .map(([left, gender]) =>
        Plot.text(gender, occlusionY({
          x: "year",
          y: "percent",
          text: left
            ? (d) => `${d.country} ${d.percent}%`
            : (d) => `${d.percent}% ${d.country}`,
          fill: "country",
          textAnchor: left ? "end" : "start",
          dx: left ? -3 : 3,
          radius: 5.5
        }))
      )
  ]
  })
}
```

<h1>How does technology impact the social views and aspect of European society?</h1>
<br>
<br>
<h4>In this section, we zoom in on public opinion around key societal issues—comparing results across regional blocs such as the EU27, the Eurozone, the Balkans, and Non-Euro countries.</h4>

<br><h2>Gender Equality</h2>
<br>
<h4>Hier moet nog wat uitleg bijkomen</h4>

<div class="card">
  <div class="chart-title">What percentage of citizens agrees that gender equality could...</div>
  <div class="grid-3">
    <div class="grid-item">
      <div class="slope-title">Improve outcomes of science and technology</div>
      <div class="slope-chart">${resize((width) => slope_graph(gender1, {width}))}</div>
    </div>
    <div class="grid-item">
      <div class="slope-title">Improve business profits and economy</div>
      <div class="slope-chart">${resize((width) => slope_graph(gender2, {width}))}</div>
    </div>
    <div class="grid-item">
      <div class="slope-title">Help ensure a fairer and more equal society</div>
      <div class="slope-chart">${resize((width) => slope_graph(gender3, {width}))}</div>
    </div>
  </div>
</div>

<h4>Hier moet nog wat uitleg bijkomen</h4>

<br><h2>Should technology be regulated?</h2>
<br>
<br>
QA11b

back-to-back bar chart

<br><h2>Impact of AI and automation on jobs</h2>
<br>
<br>
QA8.2 

welke graph?

<style>
h1 {
    display: inline;
}

h2 {
  display: inline;
}

h4 {
  display: inline;
  font-weight: normal;
}

.chart-title {
  font-weight: 600;
  margin-bottom: 0.5rem;
  font-size: 18px;
}

.slope-title {
  font-weight: 600;
  margin-bottom: 0.5rem;
}

.slope-chart {
  width: 500px;
  text-align: center;
}

.grid-3 {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 0rem;
  justify-items: center;
  align-items: start;
  margin-top: 2rem;
}

.grid-item {
  text-align: center;
  width: 300px;
}
</style>