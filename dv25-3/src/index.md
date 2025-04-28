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

<h4>How do Europeans perceive science and technology? What do they know, value, and expect from scientific advancements? This dataset presents the results of a wide-ranging European survey on public attitudes toward science and technology. It was conducted in 2024 and covers key themes such as knowledge and interest in science, perceptions of its impact on society, trust in scientific institutions, inclusiveness and diversity in science, and views on emerging technologies like artificial intelligence.</h4>
<br>
<br>
<h4>Alongside a general introduction, this site also features two dedicated sections for deeper exploration:
In the <b>Explore section</b>, you can interact with an EU map and a series of charts to compare country-level responses on key questions.
In the <b>Social section</b>, we take a closer look at regional differences—comparing how various country groups view regulation, gender equality in science, and the future of jobs in an AI-driven world. These interactive visualizations aim to make complex survey data accessible, engaging, and useful for anyone interested in how science is perceived across Europe.
<br>
<br>
<h4>Explore the data to see how perspectives differ by age, education, and region—and discover what the numbers say about the future relationship between science and society in Europe.</h4>

<br><h2>How well is the European citizen informed?</h2>
<br>
<h4>This stacked bar chart shows how well-informed citizens feel about different science and technology topics. Each bar represents a topic and is divided into 3 categories summing up to 100%.</h4>
<br>
<h4>The dotted line at 50% helps you quickly see whether the majority of people feel at least somewhat informed about a topic—or not. Hover over each segment for exact percentages.</h4>

<div class="card">
  <div class="chart-title">For each of the following, please indicate whether you are...</div>
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
<h4>The data reveals clear differences in how informed citizens feel about various societal and scientific topics. Overall, there is a trend suggesting that individuals feel less informed about scientific and technological matters than about more commonly discussed subjects like politics or sports.</h4>
<br>
<br>
<h4><b>Medical discoveries</b> and <b>scientific or technological developments</b> are areas where citizens report feeling the least informed. Nearly half of respondents (48%) feel poorly informed about medical breakthroughs, and 44% say the same about scientific and technological developments. Only 10–11% feel very well informed in these categories. This indicates a significant gap in public engagement or accessibility to information in the fields most closely tied to innovation and public health.</h4>
<br>
<br>
<h4>In contrast, <b>sports news</b> and <b>culture and arts</b> evoke stronger feelings of being informed. These topics show higher percentages in the very well informed category, with 26% for sports and 12% for culture, suggesting that people may have easier access to or more interest in these areas through media and daily conversations.</h4>
<br>
<br>
<h4><b>Politics</b> and <b>environmental issues</b>, which frequently dominate public discourse, show a relatively more balanced spread. A majority of citizens report being moderately well-informed (51–56%), and around a quarter feel very well informed. This suggests that while people may engage with these topics, the complexity or polarized nature of political and environmental discussions might limit a broader sense of deep understanding.</h4>
<br>
<br>
<h4>The results underline a need for improved science communication, particularly in the fields of medicine and technology, where innovation directly affects public well-being. Making complex topics more accessible and engaging through trustworthy sources could help bridge the current knowledge gap and empower citizens to better navigate societal changes driven by scientific advancements.</h4>

<br><h2>How do European citizens gather information?</h2>
<br>
<h4>This radar chart shows the main sources of information used by different age groups to stay up to date with science and technology. Each axis represents a different source and the lines show how frequently each age group relies on them.</h4>
<br>
<h4>The chart makes it easy to compare patterns across age groups and spot generational differences in information habits.</h4>

<div class="card">
  <div class="chart-title">What are the two main sources that you use the most to stay up to date?</div>
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
<h4>The data highlights distinct generational differences in how people stay informed about science and related topics.</h4>
<br>
<br>
<h4><b>Television</b> remains the dominant source across all age groups, especially among older citizens—69% of those aged 55–64 rely on TV, compared to 50% of those aged 15–24. Newspapers follow a similar trend, with usage increasing with age.</h4>
<br>
<h4>In contrast, younger people rely far more on <b>online social networks and blogs</b>, a stark difference from older age groups. This demographic is also more likely to consult online encyclopaedias, indicating a preference for digital, accessible platforms.</h4>
<br>
<h4><b>Books and magazines</b> show moderate use across all groups, while <b>scientific journals</b> remain niche, particularly among younger and older groups. Interestingly, <b>radio and podcasts</b> have relatively low but consistent engagement, regardless of age.</h4>
<br>
<br>
<h4>Overall, the results suggest a clear digital divide, with younger individuals gravitating toward informal, online sources, while older groups lean heavily on traditional media. These patterns underscore the importance of tailoring science communication to the platforms most trusted and accessed by each age group.</h4>

<br><h2>Trust in scientific development with the use of AI</h2>
<br>
<h4>This waffle chart illustrates how citizens with different levels of education perceive the role of AI in advancing scientific discoveries. Each level indicates the highest obtained degree of the individual's parents and is represented by a separate chart. The colored blocks show the distribution of agreement levels with the statement, from strong disagreement to strong agreement, making it easy to see how opinions vary by education level. Each block represents 1% of the group’s responses.</h4>

<div class="card">
  <div class="chart-title">Agreement with AI Advancing Scientific Progress, by Parental Education Level</div>
  <div class="grid-3">
    <div class="grid-item">
      <div class="waffle-title">Primary education (max both parents)</div>
      <div class="waffle-chart">${waffle(waffle_data1, 7828, { width: 300, height: 300 })}</div>
    </div>
    <div class="grid-item">
      <div class="waffle-title">Secondary education (max both parents)</div>
      <div class="waffle-chart">${waffle(waffle_data2, 2439, { width: 300, height: 300 })}</div>
    </div>
    <div class="grid-item">
      <div class="waffle-title">At least one parent with higher education</div>
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

<h4>The data reveals a clear correlation between educational background and positive perceptions of AI’s role in scientific progress.</h4>
<br>
<br>
<h4>Respondents with at least one parent having higher education show the strongest agreement, with more than half expressing positive views. The proportion of strong disagreement is relatively low in this group.</h4>
<br>
<h4>Those with at most secondary education are slightly less enthusiastic but still show a favorable lean, with a significant share agreeing, though neutrality is more common.</h4>
<br>
<h4>Survey participants whose parents had at most a primary education express more uncertainty and skepticism, with a higher proportion of 'Don't know', although the percentage of 'Totally disagree' remains exactly the same across all 3 levels.</h4>
<br>
<br>
<h4>Overall, <b>confidence in AI’s benefits for science appears to rise with educational attainment</b>, suggesting that familiarity with scientific or technological environments may enhance trust in the usage of AI.</h4>

<style>
.waffle-chart {
  width: 550px;
  text-align: center;
}

.waffle-title {
  font-weight: 600;
  margin-bottom: 0.5rem;
  margin-right: 1rem;
}

.chart-title {
  font-weight: 600;
  margin-bottom: 0.5rem;
  font-size: 18px;
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

h4 {
  display: inline;
  font-weight: normal;
}

</style>