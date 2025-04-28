---
title: Detailed
toc: false
---

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
const params = new URLSearchParams(window.location.search);
const selectedCountry = params.get("country") ?? "FR";

const barPlots = (async () => {
  const sentimentOrder = [
    "very negative",
    "fairly negative",
    "no effect",
    "fairly positive",
    "very positive",
    "don't know"
  ];

const plots = await Promise.all(
    Object.entries(q6Attachments).map(async ([key, value]) => {
        const data = await value;
        const filteredData = data.filter(row => row.country === selectedCountry);
        if (filteredData.length > 0) {
            const row = filteredData[0];

            const entries = sentimentOrder
                .map(k => ({ x: k, y: row[k] }))
                .filter(d => d.y !== undefined);

            return html`<div style="margin-bottom: 2rem;">
                <h3>${key}</h3>
                ${Plot.plot({
                    marks: [
                        Plot.barY(entries, { x: "x", y: "y" })
                    ],
                    x: { label: null, domain: sentimentOrder },
                    y: { label: "Count" }
                })}
            </div>`;
        }
    })
);

return html`<div style="display: flex; flex-direction: column; gap: 1rem;">
    ${plots.filter(Boolean)}
</div>`;
})();
```

<div class="card" style="display: flex; flex-direction: column; gap: 1rem;">
    The counted awnsers per topic for ${selectedCountry}:
    ${resize((width) => barPlots)} 
</div>