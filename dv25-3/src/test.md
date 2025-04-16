---
title: test
---

<div class="card">
  <h3>Population Density Map</h3>
  <script type="module">
    import("https://cdn.jsdelivr.net/npm/eurostat-map@4.1.39/+esm").then(({ default: eurostatmap }) => {
      eurostatmap
        .map("ch")
        .title("Population density in Europe")
        .stat({ eurostatDatasetCode: "demo_r_d3dens", unitText: "people/km²" })
        .legend({ x: 500, y: 180, title: "Density, people/km²" })
        .build(document.getElementById("eurostat-map"));
    }).catch(console.error);
    </script>
</div>