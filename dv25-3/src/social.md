---
title: Social views of technology
toc: false
---

```js
const percentage = await FileAttachment("data/qa11b.csv").csv({typed: true});
const informDomain = [
  "Regulated by government",
  "Free to the market"
];

function stacked_bar(percentage, {width, height} = {}){
  console.log(percentage);
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
        title: (iets, _) => iets.percentage.toFixed(2) + "%",
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
<h4>In this section, we zoom in on public opinion around key societal issues—comparing results across regional blocs such as the <b>EU27</b>, the <b>Eurozone</b>, the <b>Balkans Candidate</b> and <b>Non-Euro countries</b>.</h4>

<br><h2>How do citizens view gender equality in relation to science, the economy, and fairness?</h2>
<br>
<h4>This slope graph highlights the change in public opinion between 2021 and 2024 across the different European regions. It visualizes the percentage of people who agree that gender equality can improve science and technology outcomes, benefit the economy, and contribute to a fairer society.</h4>

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

<h4>Based on the data presented in the slope graphs, we observe a generally <b>positive perception of gender equality</b> across all European regions. However, there are slight <b>decreases</b> in agreement levels from 2021 to 2024 across most indicators.</h4>
<ul>
  <li>Belief in gender equality improving science and technology dropped modestly in most regions, especially in the EU27 and Eurozone.</li>
  <li>The view that gender equality boosts business and the economy also saw a small decline, most notably outside the Eurozone.</li>
  <li>Confidence in gender equality's role in creating a fairer society remains the highest overall, though even this saw a minor dip in the EU27 and Eurozone.</li>
</ul>
<h4>These small downward shifts may reflect growing societal tensions, economic uncertainty, or evolving political narratives around equity and inclusion. Despite the declines, support remains relatively strong, especially in the belief that gender equality promotes fairness—indicating it is still viewed as a cornerstone for societal progress. Continued public engagement and education on the benefits of equality may help maintain or even reverse this trend in future years.</h4>

<br><h2>Should technology be regulated?</h2>
<br>
<h4>Public opinion on how technology should be governed reveals a meaningful divide across different European regions. While innovation and market freedom are often celebrated as drivers of progress, there's a growing recognition of the need for regulation to ensure technology aligns with public interests, ethical standards, and social responsibility.</h4>

<div class="card">
  <div class="chart-title">Public Opinion on Technology Regulation</div>
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
<h4>The stacked bar chart presents the share of citizens who believe technology should be regulated by government versus those who prefer it to remain free to the market. The data spans the four regional blocs.</h4>
<ul>
  <li><b>Regulation is generally favored</b> across all regions, with support highest in the Balkans Candidate Countries (59.2%) and the Eurozone (57.3%). This suggests a strong public desire for oversight in regions with deeper EU integration or aspirations.</li>
  <li><b>Non-Euro countries</b> are the only group where a slight <b>majority prefers market freedom</b> (50.6%), possibly reflecting different economic priorities or political attitudes toward state intervention.</li>
  <li>The <b>EU27 average</b> sits at 55.5% in favor of regulation, reflecting a cautious but consistent European consensus that technological progress should be guided by public institutions.</li>
</ul>
<h4>Overall, the data illustrates that <b>most Europeans lean toward government regulation of technology</b>, highlighting widespread concerns about privacy, misinformation, AI risks, or corporate power. However, regional variation also suggests that historical, political, and economic contexts continue to shape public attitudes.</h4>

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

ul {
  max-width: 100%;
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