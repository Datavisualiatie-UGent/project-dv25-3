---
title: Explore
toc: false
---


<script src="https://cdnjs.cloudflare.com/ajax/libs/d3/3.5.17/d3.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/topojson/3.0.2/topojson.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/datamaps@0.5.9/dist/datamaps.world.min.js"></script>

```js
const countryCodeMapping = {
  BE: "BEL", // Belgium
  BG: "BGR", // Bulgaria
  CZ: "CZE", // Czech Republic
  DK: "DNK", // Denmark
  "D-W": "DEU", // Germany
  DE: "DEU", // Germany
  "D-E": "DEU", // Germany (East)
  EE: "EST", // Estonia
  IE: "IRL", // Ireland
  EL: "GRC", // Greece
  ES: "ESP", // Spain
  FR: "FRA", // France
  HR: "HRV", // Croatia
  IT: "ITA", // Italy
  CY: "CYP", // Cyprus
  LV: "LVA", // Latvia
  LT: "LTU", // Lithuania
  LU: "LUX", // Luxembourg
  HU: "HUN", // Hungary
  MT: "MLT", // Malta
  NL: "NLD", // Netherlands
  AT: "AUT", // Austria
  PL: "POL", // Poland
  PT: "PRT", // Portugal
  RO: "ROU", // Romania
  SI: "SVN", // Slovenia
  SK: "SVK", // Slovakia
  FI: "FIN", // Finland
  SE: "SWE", // Sweden
  TR: "TUR", // Turkey
  MK: "MKD", // North Macedonia
  ME: "MNE", // Montenegro
  RS: "SRB", // Serbia
  AL: "ALB", // Albania
  UK: "GBR", // United Kingdom
  BA: "BIH", // Bosnia and Herzegovina
  XK: "XKX" // Kosovo
};

const reversedMapping = {};

for (const [key, value] of Object.entries(countryCodeMapping)) {
  if (key === "D-E" || key === "D-W") {
    continue; // map doesnt support split germany
  }
  reversedMapping[value] = key;
}
```

```js
//Laad de files
const q6Attachments = {
  "Artificial Intelligence": FileAttachment("data/q6_Artificial Intelligence .csv").csv({typed: true}),
  "Biotechnology and genetic engineering": FileAttachment("data/q6_Biotechnology and genetic engineering.csv").csv({typed: true}),
  "Brain and cognitive enhancement": FileAttachment("data/q6_Brain and cognitive enhancement.csv").csv({typed: true}),
  "Information and Communication Technology": FileAttachment("data/q6_Information and Communication Technology.csv").csv({typed: true}),
  "Nanotechnology": FileAttachment("data/q6_Nanotechnology.csv").csv({typed: true}),
  "Nuclear energy for energy production": FileAttachment("data/q6_Nuclear energy for energy production .csv").csv({typed: true}),
  "Renewable energies": FileAttachment("data/q6_Renewable energies.csv").csv({typed: true}),
  "Space exploration": FileAttachment("data/q6_Space exploration .csv").csv({typed: true}),
  "Vaccines and combatting infectious diseases": FileAttachment("data/q6_Vaccines and combatting infectious diseases .csv").csv({typed: true})
};

for (const [key, value] of Object.entries(q6Attachments)) {
  value.then((data) => {
    console.log(`${key}: Loaded OK`);
  }).catch((error) => {
    console.error(`${key}: Failed to load`, error);
  });
}

//Form om topic te selecten
const formTopic = Inputs.select(Object.keys(q6Attachments), {unique: true, sort: true, label: "Topic:"});
```

```js
//View zodat update
const topic = view(formTopic);
```

```js
//Wacht op de async
const data = await q6Attachments[topic];
```

```js
//Zet de data om naar procent positief voor de europa plot
const mapData = {};
let min = 100;
let max = 0;
data.forEach(row => {
  const code = countryCodeMapping[row.country];
  const total = +row.total;
  const positive = +row["total positive"];
  const percent = +(positive / total * 100).toFixed(2);
  if (percent > max) {
    max = percent;
  }
  if (percent < min) {
    min = percent;
  }

  if (percent !== null) {
    mapData[code] = {
      fillKey: code,
      value: percent
    };
  }
});

//Kleurfunctie voor de kaart
let paletteScale = d3.scaleSequential(d3.interpolateViridis)
  .domain([min, max]);
const fills = {
  defaultFill: "#D3D3D3"
};
for (const key in mapData) {
  fills[key] = paletteScale(mapData[key].value);
}
```

```js
//Helper functie om legende bij kaart te tekenen
function createLegend(min, max, paletteScale) {
  const width = 50;
  const height = 300;
  const steps = 10;

  const legendSvg = d3.create("svg")
    .attr("width", width)
    .attr("height", height + 40); 

  const gradientId = "legend-gradient";
  const defs = legendSvg.append("defs");
  const gradient = defs.append("linearGradient")
    .attr("id", gradientId)
    .attr("x1", "0%").attr("y1", "100%")
    .attr("x2", "0%").attr("y2", "0%");

  for (let i = 0; i <= steps; i++) {
    const t = i / steps;
    const value = min + (max - min) * t;
    gradient.append("stop")
      .attr("offset", `${t * 100}%`)
      .attr("stop-color", paletteScale(value));
  }

  legendSvg.append("rect")
    .attr("x", 10)
    .attr("y", 10)
    .attr("width", 20)
    .attr("height", height)
    .style("fill", `url(#${gradientId})`);

  legendSvg.append("text")
    .attr("x", 5)
    .attr("y", height + 17)
    .attr("font-size", "12px")
    .attr("alignment-baseline", "middle")
    .text(`${min}%`);

  legendSvg.append("text")
    .attr("x", 5)
    .attr("y", 6)
    .attr("font-size", "12px")
    .attr("alignment-baseline", "middle")
    .text(`${max}%`);

  return legendSvg.node();
}

const legend = createLegend(min,max,paletteScale);
```

```js
const sentimentOrder = [ //Volgorde waarin we de bars willen
  "very negative",
  "fairly negative",
  "no effect",
  "fairly positive",
  "very positive",
  "don't know"
];
const detailedPlotContainer = document.createElement("div");

//Helper functie die bar chart toont als op een land klikt
function updateDetailedPlot(selectedCountry) {
  if (selectedCountry === undefined) {
    return html`<div><\div>`
  }
  const filteredData = data.filter(row => row.country === selectedCountry);
  if (filteredData.length > 0) {
    const row = filteredData[0];
    const entries = sentimentOrder
      .map(k => ({ x: k, y: row[k] }))
      .filter(d => d.y !== undefined);

    detailedPlotContainer.innerHTML = "";
    detailedPlotContainer.appendChild(html`<h3>${selectedCountry}</h3>`);
    detailedPlotContainer.appendChild(
    Plot.plot({
      marks: [Plot.barY(entries, { x: "x", y: "y" })],
      x: { label: null, domain: sentimentOrder },
      y: { label: "Count" }
    })
  );
  }
}
```

```js
let selectedCountry = undefined;
let countryTitle = "";

//Map plotter functie, bind de map aan een container div
function createMap(mapData) {
  const container = document.getElementById('container');
  container.innerHTML = '';
  var map = new Datamap({
    element: document.getElementById('container'),
    fills: fills,
    data: mapData,
    setProjection: function(element) {
      var projection = d3.geoEquirectangular()
        .center([20, 50]) //Al dit gepruts om op europa te zoomen, vermoeden dat gemaakt door amerikanen
        .scale(400)
        .translate([element.offsetWidth / 2, element.offsetHeight / 2]);
      var path = d3.geoPath()
        .projection(projection);

      return {path: path, projection: projection};
    },
    geographyConfig: {
      popupTemplate: function(geo, data) {
        return `<div class="hoverinfo">
          ${geo.properties.name}: ${data ? data.value + "%" : "N/A"}
        </div>`;
      }
    },
    done: function(datamap) {
      datamap.svg.selectAll('.datamaps-subunit').on('click', function(geography) {
        console.log("Clicked:", geography.id, geography.properties.name); //ONCLICK
        if (reversedMapping[geography.id]){
          countryTitle = geography.properties.name;
          selectedCountry = reversedMapping[geography.id];
          updateDetailedPlot(selectedCountry);
        }
      });
    }
  });
}
createMap(mapData);
```




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
<br><h2>Europe interactive map</h2>
<br>
<br>
<h4>To make the insights more interactive and comparable, we've added a dedicated Explore section where you can dive into country-level responses across a range of key questions. Start by using the EU map with a dropdown menu to select the questions you're most interested in—then instantly see how each country responds. This map provides a powerful overview of geographical patterns and national differences. You can then click a country to go to a more detailed page.</h4>

<!-- resize trick om te zorgen dat js pas laad als deze html geladen  -->
<div class="card" style="display: flex; flex-direction: column; gap: 1rem;">
  <div id="form-topic-slot"> 
    ${resize((width) => formTopic)} 
  </div>
  <div>
    <h3>Percentage of positive votes per country for this topic</h3>
  </div>

  <div style="display: flex; flex-direction: row; align-items: flex-start; gap: 1rem;">
    <div id="container" style="position: relative; width: 500px; height: 300px;"></div>
    <div>
      ${legend}
    </div>
  </div>
  ${resize((width) => detailedPlotContainer)}
</div>

<br>
<h2>Bubble chart met alle landen: internet usage everyday?</h2>
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