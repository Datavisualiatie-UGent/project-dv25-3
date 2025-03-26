---
theme: dashboard
title: Example dashboard
toc: false
---

# Rocket launches 🚀

<!-- Load and transform the data -->

```js
const launches = FileAttachment("data/launches.csv").csv({typed: true});
const data = await FileAttachment("data/radar_chart.csv").csv({typed: true});
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

<!-- Cards with big numbers -->

<div class="grid grid-cols-4">
  <div class="card">
    <h2>United States 🇺🇸</h2>
    <span class="big">${launches.filter((d) => d.stateId === "US").length.toLocaleString("en-US")}</span>
  </div>
  <div class="card">
    <h2>Russia 🇷🇺 <span class="muted">/ Soviet Union</span></h2>
    <span class="big">${launches.filter((d) => d.stateId === "SU" || d.stateId === "RU").length.toLocaleString("en-US")}</span>
  </div>
  <div class="card">
    <h2>China 🇨🇳</h2>
    <span class="big">${launches.filter((d) => d.stateId === "CN").length.toLocaleString("en-US")}</span>
  </div>
  <div class="card">
    <h2>Other</h2>
    <span class="big">${launches.filter((d) => d.stateId !== "US" && d.stateId !== "SU" && d.stateId !== "RU" && d.stateId !== "CN").length.toLocaleString("en-US")}</span>
  </div>
</div>

<!-- Plot of launch history -->

```js
function launchTimeline(data, {width} = {}) {
  return Plot.plot({
    title: "Launches over the years",
    width,
    height: 300,
    y: {grid: true, label: "Launches"},
    color: {...color, legend: true},
    marks: [
      Plot.rectY(data, Plot.binX({y: "count"}, {x: "date", fill: "state", interval: "year", tip: true})),
      Plot.ruleY([0])
    ]
  });
}
```

<div class="grid grid-cols-1">
  <div class="card">
    ${resize((width) => launchTimeline(launches, {width}))}
  </div>
</div>

<!-- Plot of launch vehicles -->

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

<div class="grid grid-cols-1">
  <div class="card">
    ${resize((width) => vehicleChart(launches, {width}))}
  </div>
</div>

Data: Jonathan C. McDowell, [General Catalog of Artificial Space Objects](https://planet4589.org/space/gcat)

```js
function radar_chart(data, {width, height} = {}) {
  const points = data.flatMap(({ Age, ...values }) => Object.entries(values).map(([key, value]) => ({ Age, key, value })))
  const longitude = d3.scalePoint(new Set(Plot.valueof(points, "key")), [180, -180]).padding(0.5).align(1)

  return Plot.plot({
      width:700,
      height:700,
      marginTop: 30,
      projection: {
      type: "azimuthal-equidistant",
      rotate: [0, -90],
      domain: d3.geoCircle().center([0, 90]).radius(0.8)()
      },
      color: { legend: true },
      marks: [
      // grey discs
      Plot.geo([0.7, 0.6, 0.5, 0.4, 0.3, 0.2, 0.1], {
          geometry: (r) => d3.geoCircle().center([0, 90]).radius(r)(),
          stroke: "white",
          fill: "white",
          strokeOpacity: 0.3,
          fillOpacity: 0.03,
          strokeWidth: 0.5
      }),
  
      // white axes
      Plot.link(longitude.domain(), {
          x1: longitude,
          y1: 90 - 0.73,
          x2: 0,
          y2: 90,
          stroke: "white",
          strokeOpacity: 0.5,
          strokeWidth: 3.5
      }),
  
      // percentage labels
      Plot.text([0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7], {
          x: 180,
          y: (d) => 90 - d,
          dx: 2,
          textAnchor: "start",
          text: (d) => `${100 * d}%`,
          fill: "black",
          stroke: "white",
          fontSize: 12
      }),
  
      // axes labels
      Plot.text(longitude.domain(), {
          x: longitude,
          y: 90 - 0.79,
          text: Plot.identity,
          lineWidth: 5,
          fontSize: 16
      }),
  
      // areas
      Plot.area(points, {
          x1: ({ key }) => longitude(key),
          y1: ({ value }) => 90 - value,
          x2: 0,
          y2: 90,
          fill: "Age",
          stroke: "Age",
          curve: "cardinal-closed"
      }),
  
      // points
      Plot.dot(points, {
          x: ({ key }) => longitude(key),
          y: ({ value }) => 90 - value,
          fill: "Age",
          stroke: "white"
      }),
  
      // interactive labels
      Plot.text(
          points,
          Plot.pointer({
          x: ({ key }) => longitude(key),
          y: ({ value }) => 90 - value,
          text: (d) => `${(100 * d.value).toFixed(0)}%`,
          textAnchor: "start",
          dx: 4,
          fill: "black",
          stroke: "white",
          maxRadius: 10
          })
      ),
  
      // interactive opacity on the areas
      () =>
          svg`<style>
              g[aria-label=area] path {fill-opacity: 0.1; transition: fill-opacity .2s;}
              g[aria-label=area]:hover path:not(:hover) {fill-opacity: 0.05; transition: fill-opacity .2s;}
              g[aria-label=area] path:hover {fill-opacity: 0.3; transition: fill-opacity .2s;}
          `
      ]
  });
}
```

<div class="grid grid-cols-1">
  <div class="card">
    ${resize((width) => radar_chart(data, {width}))}
  </div>
</div>

