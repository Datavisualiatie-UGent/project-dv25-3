---
title: Explore
toc: false
---

```js
const launches = FileAttachment("data/launches.csv").csv({typed: true});
```

<!-- A shared color scale for consistency, sorted by the number of launches -->

```js
const color = Plot.scale({
  color: {
    type: "categorical",
    domain: d3.groupSort(launches, (D) => -D.length, (d) => d.state).filter((d) => d !== "Other"),
    unknown: "var(--theme-foreground-muted)"
  }
});
```

```js
function vehicleChart(data, {width}) {
  return Plot.plot({
    title: "Popular launch vehicles",
    width,
    height: 300,
    marginTop: 0,
    marginLeft: 50,
    x: {grid: true, label: "Launches"},
    y: {label: null},
    color: {...color, legend: true},
    marks: [
      Plot.rectX(data, Plot.groupY({x: "count"}, {y: "family", fill: "state", tip: true, sort: {y: "-x"}})),
      Plot.ruleX([0])
    ]
  });
}
```
<h1>Explore the survey</h1>
<br><h2>Europa interactieve kaart</h2>
<br>
<br>
<h4>To make the insights more interactive and comparable, we've added a dedicated Explore section where you can dive into country-level responses across a range of key questions. Start by using the EU map with a dropdown menu to select the questions you're most interested in—then instantly see how each country responds. This map provides a powerful overview of geographical patterns and national differences.</h4>

kaart komt hier:

<br>
<h2>Bubble chart met alle landen: internet usage everyday</h2>
<br>
<br>
<h4>Curious about digital habits? Check out our bubble chart, where each country is represented by a circle sized by population and colored by the percentage of people who use the internet daily. It's a simple, visual way to grasp how digital engagement varies across Europe.</h4>

plot vervangen:

<div class="grid grid-cols-1">
  <div class="card">
    ${resize((width) => vehicleChart(launches, {width}))}
  </div>
</div>

<style>
h2 {
  display: inline;
}

h4 {
  display: inline;
  font-weight: normal;
}
</style>