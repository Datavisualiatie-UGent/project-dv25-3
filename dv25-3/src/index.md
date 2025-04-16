---
title: Introduction
toc: false
---
```js
const percentage = await FileAttachment("data/informed.csv").csv({typed: true});
const informDomain = [
  "Poorly informed",
  "Moderately well-informed",
  "Very well informed"
];

function stacked_bar(percentage, {width, height} = {}){
  const firstCategory = percentage.columns[1];
  const sortedSubjects = [...percentage].sort((a, b) => d3.ascending(a[firstCategory], b[firstCategory])).map(d => d.name);
  const tidy = percentage.columns.slice(1).flatMap(percent => percentage.map(d => ({subject: d.name, percent, percentage: d[percent]})));

  return Plot.plot({
    width: 928,
    height: 600,
    marginLeft: 400,
    marginTop: 40,
    marginBottom: 20,
    style: {fontSize: "16px"},
    y: {label: null, domain: sortedSubjects},
    x: {axis: "top", tickFormat: d => `${d}%`, tickSize: 0},
    color: {scheme: "Blues"},
    marks: [
      Plot.barX(tidy, {
        y: "subject",
        x: "percentage",
        title: (iets, _) => iets.percentage + "%",
        fill: "percent",
        sort: {color: null}
      }),
      Plot.ruleX([50], {
        stroke: "black",
        strokeDasharray: "4,4",
        strokeWidth: 1.5
      }),
      Plot.text([50], {
        text: "50%",
        x: sortedSubjects[0],
        textAnchor: "middle",
      }),
      () =>
          svg`<style>
              g[aria-label=bar] rect {fill-opacity: 0.8; transition: fill-opacity .2s; cursor: pointer}
              g[aria-label=bar]:hover rect:not(:hover) {fill-opacity: 0.3;}
              g[aria-label=bar] rect:hover {fill-opacity: 1;}
              `
    ]
  });
}
```


```js
const data = await FileAttachment("data/radar_chart.csv").csv({typed: true});
const ageDomain = [
  "Age: 15-24",
  "Age: 35-44",
  "Age: 55-64",
];
const usedColors = ["#4269d0","#efb118","#ff725c","#6cc5b0","#3ca951","#ff8ab7","#a463f2","#97bbf5","#9c6b4e","#9498a0"];
function radar_chart(data, {width, height} = {}) {
  const points = data.flatMap(({ Age, ...values }) => Object.entries(values).map(([key, value]) => ({ Age, key, value })))
  const longitude = d3.scalePoint(new Set(Plot.valueof(points, "key")), [180, -180]).padding(0.5).align(1)

  return Plot.plot({
      width:1060,
      height:700,
      marginTop: 30,
      projection: {
      type: "azimuthal-equidistant",
      rotate: [0, -90],
      domain: d3.geoCircle().center([0, 90]).radius(0.8)()
      },
      color: {
        type: "ordinal",
        range: usedColors
       },
      marks: [
      // grey discs
      Plot.geo([0.7, 0.6, 0.5, 0.4, 0.3, 0.2, 0.1], {
          geometry: (r) => d3.geoCircle().center([0, 90]).radius(r)(),
          stroke: "white",
          fill: "black",
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
          stroke: "black",
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

```js
const waffle_data1 = d3.csvParse(`group,freq
Don't know,548
Totally disagree,319
Tend to disagree,1076
Neither agree nor disagree,2485
Tend to agree,2884
Totally agree,508`)

const waffle_data2 = d3.csvParse(`group,freq
Don't know,116
Totally disagree,74
Tend to disagree,285
Neither agree nor disagree,719
Tend to agree,1048
Totally agree,197`)

const waffle_data3 = d3.csvParse(`group,freq
Don't know,80
Totally disagree,132
Tend to disagree,485
Neither agree nor disagree,1207
Tend to agree,2058
Totally agree,604`)

const groupDomain = [
  "Don't know",
  "Totally disagree",
  "Tend to disagree",
  "Neither agree nor disagree",
  "Tend to agree",
  "Totally agree"
];
const colorRange = ["#888888", ...d3.schemeRdYlGn[groupDomain.length - 1]];

function waffle(w_data, total, {width, height} = {}) {
  const units = w_data.flatMap(d => d3.range(Math.round((d.freq / total) * 100)).map(() => d));
  
  return Plot.plot({
    marks: [
      Plot.cell(
        units,
        Plot.stackX({
          y: (_, i) => i % 10,
          fill: "group",
          title: (iets, _) => Math.round((iets.freq / total) * 100) + "%"
        })
      ),
      Plot.text(
        units,
        Plot.stackX({
          y: (_, i) => i % 5,
          text: "",
        })
      ),
      () =>
        svg`<style>
            g[aria-label=cell] rect {fill-opacity: 0.8; transition: fill-opacity .2s; cursor: pointer}
            g[aria-label=cell]:hover rect:not(:hover) {fill-opacity: 0.3;}
            g[aria-label=cell] rect:hover {fill-opacity: 1;}
        `
    ],
    x: { axis: null },
    y: { axis: null , reverse: true},
    color: {
      type: "ordinal",
      domain: groupDomain,
      range: colorRange
    }
  });
}
```
<div class="hero">
  <h1>European citizens: knowledge and attitudes towards science and technology</h1>
</div>

Introduction about the survey and the dataset
<br>
<br>

## How well is the European citizen informed?

<br>Uitleg ervoor

<div class="card">
  <div class="waffle-title">For each of the following, please indicate if you are...</div>
  <div class="mt-4">
        ${Plot.legend({
          color: {
            type: "ordinal",
            domain: informDomain,
            scheme: "Blues" 
          },
          columns: 4,
          style: {
              fontSize: "14px",
              spacing: "0.5rem"
            }
        })}
  </div>
  ${resize((width) => stacked_bar(percentage, {width}))}
</div>
Uitleg erna
<br>
<br>
<br>

## How do European citizens gather information?

<br>Uitleg ervoor

<div class="card">
  <div class="waffle-title">What are the two main sources that you use the most to stay up to date?</div>
  <div class="grid">
    <div class="mt-4">
      ${Plot.legend({
        color: {
          type: "ordinal",
          domain: ageDomain,
          range: usedColors
        },
        columns: 6,
        style: {
          fontSize: "14px"
        }
      })}
    </div>
  </div>
  <div class="grid grid-cols-1">
      ${resize((width) => radar_chart(data, {width}))}
  </div>
  <div class="waffle-title"> </div>
</div>

Uitleg erna
<br>
<br>
<br>

## Trust in scientific development with the use of AI

<br>Uitleg: vooral over de data
<div class="card">
  <div class="waffle-title">'AI used in science advances scientific discoveries that will lead to solutions to major challenges such as climate change and serious diseases'?</div>
  <div class="grid-3">
    <div class="grid-item">
      <div class="waffle-title">Primary maximum</div>
      <div class="waffle-chart">${waffle(waffle_data1, 7828, { width: 300, height: 300 })}</div>
    </div>
    <div class="grid-item">
      <div class="waffle-title">Secondary maximum</div>
      <div class="waffle-chart">${waffle(waffle_data2, 2439, { width: 300, height: 300 })}</div>
    </div>
    <div class="grid-item">
      <div class="waffle-title">One or two higher education</div>
      <div class="waffle-chart">${waffle(waffle_data3, 4566, { width: 300, height: 300 })}</div>
    </div>
  </div>
  <div class="grid">
    <div class="mt-4">
      ${Plot.legend({
        color: {
          type: "ordinal",
          domain: groupDomain,
          range: colorRange
        },
        columns: 4,
        style: {
            fontSize: "14px",
            spacing: "0.5rem"
          }
      })}
    </div>
  </div>
</div>

Vervolg van de uitleg: wat we kunnen zien en afleiden uit de mooie visualisatie


<style>
.waffle-chart {
  width: 550px;
  text-align: center;
}

.waffle-title {
  font-weight: 600;
  margin-bottom: 0.5rem;
}

.hero {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: var(--sans-serif);
  margin: 1rem 0 3rem;
  text-wrap: balance;
  text-align: center;
}

.hero h1 {
  padding: 0.5rem 0;
  max-width: none;
  font-size: 14vw;
  font-weight: 700;
  line-height: 1;
  background: linear-gradient(30deg, var(--theme-foreground-focus), currentColor);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

@media (min-width: 640px) {
  .hero h1 {
    font-size: 70px;
  }
}

.grid-3 {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 2rem;
  justify-items: center;
  align-items: start;
  margin-top: 2rem;
}

.grid-item {
  text-align: center;
  width: 300px;
}

h2 {
  display: inline;
}
</style>