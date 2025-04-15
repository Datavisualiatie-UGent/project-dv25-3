---
title: Introduction
toc: false
---

<div class="hero">
  <h1>Datavisualisatie project</h1>
  <h2>Welcome to your new app! Edit&nbsp;<code style="font-size: 90%;">src/index.md</code> to change this page.</h2>
</div>
hier komt de how well informed voor elk van de 6 categorieën
<div class="card">${
  resize((width) => Plot.plot({
    title: "How big are penguins, anyway? 🐧",
    width,
    grid: true,
    x: {label: "Body mass (g)"},
    y: {label: "Flipper length (mm)"},
    color: {legend: true},
    marks: [
      Plot.linearRegressionY(penguins, {x: "body_mass_g", y: "flipper_length_mm", stroke: "species"}),
      Plot.dot(penguins, {x: "body_mass_g", y: "flipper_length_mm", stroke: "species", tip: true})
    ]
  }))
}</div>

```js
const data = await FileAttachment("data/radar_chart.csv").csv({typed: true});
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
## What are the main 2 sources used to gather information by age

<div class="grid grid-cols-1">
  <div class="card">
    ${resize((width) => radar_chart(data, {width}))}
  </div>
</div>

## Trust in scientific development with the use of AI, by education level

Hier komt waffle chart

<style>

.hero {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: var(--sans-serif);
  margin: 4rem 0 8rem;
  text-wrap: balance;
  text-align: center;
}

.hero h1 {
  margin: 1rem 0;
  padding: 1rem 0;
  max-width: none;
  font-size: 14vw;
  font-weight: 900;
  line-height: 1;
  background: linear-gradient(30deg, var(--theme-foreground-focus), currentColor);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.hero h2 {
  margin: 0;
  max-width: 34em;
  font-size: 20px;
  font-style: initial;
  font-weight: 500;
  line-height: 1.5;
  color: var(--theme-foreground-muted);
}

@media (min-width: 640px) {
  .hero h1 {
    font-size: 90px;
  }
}

</style>
