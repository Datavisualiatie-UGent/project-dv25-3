---
title: Explore
toc: false
---

<script src="https://cdnjs.cloudflare.com/ajax/libs/d3/3.5.17/d3.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/topojson/3.0.2/topojson.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/datamaps@0.5.9/dist/datamaps.world.min.js"></script>

```js
const launches = FileAttachment("data/launches.csv").csv({typed: true});
```

```js
const datasetbar = FileAttachment("data/q6_Artificial Intelligence .csv").csv({typed: true});
```

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
```

```js
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

// Display the header line of each CSV to prove it worked
for (const [key, value] of Object.entries(q6Attachments)) {
  value.then((data) => {
    console.log(`${key}: Loaded OK`);
  }).catch((error) => {
    console.error(`${key}: Failed to load`, error);
  });
}

const formTopic = Inputs.select(Object.keys(q6Attachments), {unique: true, sort: true, label: "Topic:"});
```

```js
const topic = view(formTopic);
```

```js
const data = await q6Attachments[topic];
```

```js
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

let paletteScale = d3.scaleLinear([min,max], ["#EFEFFF","#02386F"]); // blue color
// Create a fills object dynamically based on the scale
const fills = {
  defaultFill: "#D3D3D3"
};
for (const key in mapData) {
  fills[key] = paletteScale(mapData[key].value);
}
```

```js
function createMap(mapData) {
  // Destroy the existing map if it exists
  const container = document.getElementById('container');
  container.innerHTML = '';
  new Datamap({
    element: document.getElementById('container'),
    fills: fills,  // Use dynamic fills here
    data: mapData,  // The mapData with `fillKey`
    geographyConfig: {
      popupTemplate: function(geo, data) {
        return `<div class="hoverinfo">
          ${geo.properties.name}: ${data ? data.value + "%" : "N/A"}
        </div>`;
      }
    },
    done: function(datamap) {
      datamap.svg.selectAll('.datamaps-subunit').on('click', function(geography) {
        console.log("Clicked:", geography.id, geography.properties.name);
      });
    }
  });
}
createMap(mapData);
```


<div class="card" style="display: flex; flex-direction: column; gap: 1rem;">
  ${formTopic}

  <div style="font-family: monospace; line-height: 1.5;">
    <h3>Percentage of positive votes per country for this topic</h3>
  </div>

  <div>
    ${topic}
  </div>

  <div id="container" style="position: relative; width: 500px; height: 300px;"></div>

</div>



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

<br>
<h2>Stacked bar chart</h2>
<br>
<br>
<h4>A stacked bar chart of people's opinion on the impact of AI in their life. the width of the bars corresponds width the amount of people from these places in the dataset.</h4>

```js
function barChart(data, {w}){
  const parseddata= data.map(d => [parseInt(d['don\'t know']),parseInt(d['very positive']),parseInt(d['fairly positive']),parseInt(d['no effect']),parseInt(d['fairly negative']),parseInt(d['very negative'])])

  const width = 2500;
  const height = 300;
  const graph = d3.select('body').append('svg').attr("width",w).attr("height",height);

  const barHeigth = 200;

  const scale = 1/36

  const buffer = 30

  const color_dontknow='#A9A9A9'
  const color_verygood='#228B22'
  const color_good='#ADFF2F'
  const color_neutral='#FFD700'
  const color_bad='#FF8C00'
  const color_verybad='#e41a1c'

  const colors = [color_dontknow,color_verygood,color_good,color_neutral,color_bad,color_verybad]
  const colordefs=["geen mening","heel positief","vrij positief","neutraal","vrij negatief","heel negatief"]

  graph.append("g")
    .selectAll("g")
    .data(parseddata)
    .enter().append("g")
      .attr("x", function(d,i) {
        var parsum=0;
        for (let j = 0; j < i; j++){
          parsum+=parseddata[j].reduce((partialSum, a) => partialSum + a, 0);
        }
        return parsum*scale+i*1;
      })
      //.attr("x", (d, i) => i * 21)
      .attr("index", (d, i) => i)
      .attr("sum", d => d.reduce((partialSum, a) => partialSum + a, 0))
      //.attr("data", (d, i) => d)
    .selectAll("rect")
    .data((d,i) => parseddata[i])
    .enter().append("rect")
      .attr("x",function() {
        return d3.select(this.parentNode).attr("x");
        })
      // .attr("x",function() {
      //   let ind=d3.select(this.parentNode).attr("index");
      //   return ind*20;
      //   })
      .attr("y",function(d,i) {
        let ind=d3.select(this.parentNode).attr("index");
        let dat=parseddata[ind].slice(0, i);
        let parsum = dat.reduce((partialSum, a) => partialSum + a, 0);
        return parsum*barHeigth/d3.select(this.parentNode).attr("sum")+buffer;
        })
      .attr("width",function(d,i){
        return d3.select(this.parentNode).attr("sum")*scale;
      })
      .attr("height",function(d,i){
        return d*barHeigth/d3.select(this.parentNode).attr("sum");
      })
      .attr("fill",(d,i) => colors[i]);



  graph.append("g")
    .selectAll("text")
    .data(parseddata)
    .enter()
    .append("text")
    .text((d,i) => data[i]["country"])
    .style("text-anchor", "middle")
    .style("font-size", "8px")
    .attr("y",buffer+barHeigth+20)
    .attr("x",function(d,i) {
      var parsum=0;
        for (let j = 0; j < i; j++){
          parsum+=parseddata[j].reduce((partialSum, a) => partialSum + a, 0);
        }
      var begin=parsum*scale+1*i;
      var sum=d.reduce((partialSum, a) => partialSum + a, 0)
      return begin+sum*scale/2;
    });

  graph.append("g")
    .selectAll("rect")
    .data(colors)
    .enter()
    .append("rect")
    .attr("x",(d,i) => 10+i*150)
    .attr("y",10)
    .attr("width",10)
    .attr("height",10)
    .attr("fill", d => d);

  graph.append("g")
    .selectAll("text")
    .data(colors)
    .enter()
    .append("text")
    .attr("x",(d,i) => 20+i*150)
    .attr("y",20)
    .text((d,i) => colordefs[i]);

  return graph.node();
}

```

<div class="grid grid-cols-1">
  <div class="card">
    ${resize((width) => barChart(data, {width}))}
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
