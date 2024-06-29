<script>
  import { onMount } from "svelte";
  import mapboxgl from "mapbox-gl";
  import { geojsonData } from "../lib/data.geojson.js";
  import { cibuburGeoJSON } from "../lib/cibubur.geojson.js"; // Data GeoJSON untuk wilayah Cibubur

  let map;
  let selectedJenisUsaha = "Semua";
  let jenisUsahaList = ["Semua"];

  geojsonData.features.forEach((feature) => {
    const jenisUsaha = feature.properties["Jenis Usaha"];
    if (jenisUsaha && !jenisUsahaList.includes(jenisUsaha)) {
      jenisUsahaList.push(jenisUsaha);
    }
  });

  onMount(() => {
    mapboxgl.accessToken =
      "pk.eyJ1IjoiZmhtaWFsZmk3OSIsImEiOiJja3ZoazI2bm4yaXR6Mm9ubzQ3aHVsMWdxIn0.JPnp5fiT7CQWK12NEp1aRg";

    map = new mapboxgl.Map({
      container: "map",
      style: "mapbox://styles/mapbox/streets-v11",
      center: [106.87613, -6.347729],
      zoom: 14.15,
    });

    map.on("load", () => {
      map.addSource("places", {
        type: "geojson",
        data: geojsonData,
      });

      map.addLayer({
        id: "places",
        type: "circle",
        source: "places",
        paint: {
          "circle-color": "#4264fb",
          "circle-radius": 6,
          "circle-stroke-width": 2,
          "circle-stroke-color": "#ffffff",
        },
      });

      // Tambahkan sumber dan lapisan untuk wilayah Cibubur
      map.addSource("cibubur", {
        type: "geojson",
        data: cibuburGeoJSON,
      });

      map.addLayer({
        id: "cibubur",
        type: "fill",
        source: "cibubur",
        layout: {},
        paint: {
          "fill-color": "#088",
          "fill-opacity": 0.5,
        },
      });

      const popup = new mapboxgl.Popup({
        closeButton: false,
        closeOnClick: false,
      });

      map.on("mouseenter", "places", (e) => {
        map.getCanvas().style.cursor = "pointer";

        const coordinates = e.features[0].geometry.coordinates.slice();
        const {
          "Pemilik Usaha": owner,
          "Nama Usaha": businessName,
          "Jenis Usaha": businessType,
          Omzet: omzet,
        } = e.features[0].properties;

        const description = `
          <strong>Pemilik Usaha:</strong> ${owner}<br>
          <strong>Nama Usaha:</strong> ${businessName}<br>
          <strong>Jenis Usaha:</strong> ${businessType}<br>
          <strong>Omzet:</strong> ${omzet}
        `;

        while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
          coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
        }

        popup.setLngLat(coordinates).setHTML(description).addTo(map);
      });

      map.on("mouseleave", "places", () => {
        map.getCanvas().style.cursor = "";
        popup.remove();
      });

      // Tambahkan kontrol navigasi untuk zoom in dan zoom out
      const nav = new mapboxgl.NavigationControl();
      map.addControl(nav, "bottom-right");
    });

    return () => {
      map.remove();
    };
  });

  function updateMap() {
    const filteredData = {
      ...geojsonData,
      features: geojsonData.features.filter(
        (feature) =>
          selectedJenisUsaha === "Semua" ||
          feature.properties["Jenis Usaha"] === selectedJenisUsaha
      ),
    };

    map.getSource("places").setData(filteredData);
  }
</script>

<div class="container">
  <div class="relative" id="map">
    <div class="filter-container absolute z-10">
      <label for="jenisUsaha">Filter Jenis Usaha:</label>
      <select
        id="jenisUsaha"
        bind:value={selectedJenisUsaha}
        on:change={updateMap}
      >
        {#each jenisUsahaList as jenisUsaha}
          <option value={jenisUsaha}>{jenisUsaha}</option>
        {/each}
      </select>
    </div>
  </div>
</div>

<style>
  .filter-container {
    position: absolute;
    top: 10px;
    left: 10px;
    z-index: 1;
    background: white;
    padding: 10px;
    border-radius: 5px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  }

  #map {
    position: absolute;
    top: 0;
    bottom: 0;
    width: 100%;
    height: 100%;
  }
</style>
