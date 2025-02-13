---
theme: dashboard
toc: false
title: Flood data
---
<!-- to make the plot span length of the window -->

# Flood data

```js
const flood = await FileAttachment("./data/preprocessed.csv").csv();
const floodjeojson = {
  type: "FeatureCollection",
  features: flood.map(event => ({
    type: "Feature",
    geometry: {
      type: "Point",
      coordinates: [+event.longitude, +event.latitude]
    }
  }))
};

const us = await FileAttachment("./data/counties-10m.json").json();
const stateID = ["25015", "50", "25", "09"];
const statesforIDs = {
  type: "GeometryCollection",
  geometries: us.objects.states.geometries.filter(geo => stateID.includes(geo.id))
};
const states = topojson.feature(us, statesforIDs);
```

<!-- ```js
function floodPlot(data, {width} = {}) {

  const stateNames = {
    "NH" : "New Hampshire",
    "VT" : "Vermont",
    "MA" : "Massachusetts",
    "CT" : "Connecticut",
  };

  const sumfloodclusters = Array.from(
    d3.rollup(
      data,
      (v) => d3.sum(v, (d) => d.flooded_cluster_default),
      (d) => d.state
    ),
    ([state, totalClusters]) => ({ state, totalClusters })
  );

  return Plot.plot({
    title: "Flood clusters by state",
    width,
    x: {type: "band", domain: sumfloodclusters.map(d => d.state), label: "State" },
    y: {grid: true, inset: 10, label: "Total Flood Clusters"},
    marks: [
      Plot.barY(sumfloodclusters, {
        x: "state",
        y: "totalClusters",
        fill: "steelblue",
        title: (d) => `${stateNames[d.state] || d.state} : ${d.totalClusters} clusters`
        }
          )
        ]
      })
  };
``` -->

<!-- ```js
function floodMap(datacsv, shapes, {width} = {}) {
  return Plot.plot({
    projection: d3.geoAlbersUsa().
    scale(5000).
    translate([-1000, 850]),
    width: width,
    marks: [
      Plot.geo(shapes),
      Plot.dot(datacsv, {
        x: "longitude",
        y: "latitude",
        r: "flooded_cluster_default",
        fill: "cyan",
        opacity: 0.6,
      })
    ]
  });
}
``` -->

```js
import maplibregl from "npm:maplibre-gl";

const map = new maplibregl.Map({
  container: "map",
  zoom: 7,
  center: [-71.1805, 43.2855],
  pitch: 52,
  hash: true,
  style: {
    version: 8,
    sources: {
      osm: {
        type: "raster",
        tiles: ["https://a.tile.openstreetmap.org/{z}/{x}/{y}.png"],
        tileSize: 256,
        attribution: "&copy; OpenStreetMap Contributors",
        maxzoom: 19
      }
    },
    layers: [
      {
        id: "osm",
        type: "raster",
        source: "osm"
      }
    ]
  },
  maxZoom: 18,
  maxPitch: 85
});

map.addControl(
  new maplibregl.NavigationControl({
    visualizePitch: true,
    showZoom: true,
    showCompass: true
  })
);

map.on("move", () => {
  const center = map.getCenter();
  document.getElementById("coordinates").innerHTML = 
  `Center: ${center.lng.toFixed(4)}, ${center.lat.toFixed(4)}`;
});

map.on("load", () => {
  map.addSource('floodPoints', {
    type: 'geojson',
    data: floodjeojson
  });
  map.addLayer({
  id: 'floodPointsLayer',
  type: 'circle',
  source: 'floodPoints',
  paint: {
    'circle-radius': 4,
      'circle-color': '#0000FF',
      'circle-opacity': 0.8
  }
});
});

// flood.forEach(event => {
//     const lng = +event.longitude;
//     const lat = +event.latitude;

//     if (!isNaN(lng) && !isNaN(lat)) {
//       new maplibregl.Marker().
//       setLngLat([lng, lat]).
//       addTo(map);
//     }
// });

```

<!-- ```js
new maplibregl.Marker()
  .setLngLat([11.39, 47.29])
  .addTo(map);
display(floodPlot(flood));
``` -->

<!-- grid makes plots resizable -->
<!-- <div class="grid grid-cols-1">
  <div class="card">${resize((width) => floodMap(flood, states, {width}))}</div>
</div> -->
<div id="map" style="width: 100%; height: 450px;"></div>
<link rel="stylesheet" href="npm:maplibre-gl/dist/maplibre-gl.css">
<script src="https://d3js.org/d3.v6.min.js"></script>

<div id="coordinates" style="position: absolute; bottom: 10px; left: 10px; background: white; padding: 5px; border-radius: 3px; z-index: 1;">
  Center: Loading...
</div>
