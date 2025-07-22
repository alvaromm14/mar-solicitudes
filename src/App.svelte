<script>
  import world from "$data/world-110m.json";
  import * as topojson from "topojson-client";
  import countryToISO from "./data/countryToISO.js";

  import { geoCentroid, geoOrthographic, geoPath, geoArea } from "d3-geo";
  import { scaleLinear } from "d3-scale";
  import { max } from "d3-array";
  import { timer } from "d3-timer";
  import { drag } from "d3-drag";
  import { select } from "d3-selection";
  import { onMount } from "svelte";
  import { spring } from "svelte/motion";

  import Glow from "$components/Glow.svelte";
  import Tooltip from "./components/Tooltip.svelte";
  import ToggleButton from "./components/ToggleButton.svelte";
  import ClaimsCountrySelector from "$components/ClaimsCountrySelector.svelte";
  import data from "$data/data.json";

  let width = 400;
  $: height = width;

  $: projection = geoOrthographic()
    .scale(width / 2)
    .rotate([$xRotation, $yRotation])
    .translate([width / 2, height / 2]);

  $: path = geoPath(projection);

  let xRotation = spring(0, { stiffness: 0.08, damping: 0.4 });
  let yRotation = spring(0, { stiffness: 0.1, damping: 0.7 });

  let degreesPerFrame = 0.5;

  const t = timer((elapsed) => {
    if (dragging || tooltipData) return;
    $xRotation += degreesPerFrame;
  }, 0);

  const DRAG_SENSITIVITY = 1;
  let dragging = false;

  onMount(() => {
    const element = select(globe);
    element.call(
      drag()
        .on("drag", (event) => {
          $xRotation = $xRotation + event.dx * DRAG_SENSITIVITY;
          $yRotation = $yRotation - event.dy * DRAG_SENSITIVITY;
          dragging = true;
        })
        .on("end", () => {
          dragging = false;
        }),
    );
  });

  let countries = topojson.feature(world, world.objects.countries).features;
  let borders = topojson.mesh(
    world,
    world.objects.countries,
    (a, b) => a !== b,
  );

  const colorScale = scaleLinear()
    .domain([0, max(data, (d) => d.population)])
    .range(["#26362E", "#0DCC6C"]);

  let globe;

  let zeeFeatures = [];
  let claimsFeatures = [];

  function smallClaims(features) {
    return features.filter((f) => f.geometry && geoArea(f) < 1.0);
  }

  onMount(async () => {
    const zeeTopo = await (await fetch("/data/zee.json")).json();

    const zeeGeo = topojson.feature(zeeTopo, zeeTopo.objects.zee);

    let features = [];

    if (zeeGeo.type === "FeatureCollection") {
      features = zeeGeo.features;
    } else if (zeeGeo.type === "GeometryCollection") {
      features = zeeGeo.geometries.map((geom) => ({
        type: "Feature",
        geometry: geom,
        properties: {},
      }));
    } else if (zeeGeo.type === "Feature") {
      features = [zeeGeo];
    }

    features.forEach((f) => {
      if (f.properties?.MRGID === 8444) {
        reverseRings(f.geometry);
      }
    });

    zeeFeatures = features.filter((f) => f.properties?.MRGID !== 8444);

    const claims = await (await fetch("/data/claims1.geojson")).json();
    claimsFeatures = smallClaims(claims.features);
  });

  function reverseRings(geom) {
    const rev = (ring) => ring.slice().reverse();
    if (geom.type === "Polygon") {
      geom.coordinates = geom.coordinates.map(rev);
    } else if (geom.type === "MultiPolygon") {
      geom.coordinates = geom.coordinates.map((poly) => poly.map(rev));
    }
  }

  let tooltipData;

  $: if (tooltipData) {
    const center = geoCentroid(tooltipData);
    $xRotation = -center[0];
    $yRotation = -center[1];
  }

  function handleToggle(active) {
    highlightECS = active;
  }
  let highlightECS = false;

  let tooltipX = 0;
  let tooltipY = 0;

  function handleClaimClick(cl, event) {
    tooltipData = cl;
    tooltipX = event.clientX;
    tooltipY = event.clientY;
  }

  let selectedClaimCountry = "";
  let selectedISOCode = "";

  function handleCountrySelect(country) {
    selectedClaimCountry = country;

    // Busca el país en 'countries' por nombre, ignorando mayúsculas y espacios
    const countryData = countries.find(
      (c) =>
        c.country &&
        c.country.trim().toLowerCase() === country.trim().toLowerCase(),
    );

    if (countryData) {
      const center = geoCentroid(countryData);

      // Asignar a springs para animación suave
      $xRotation = -center[0];
      $yRotation = -center[1];
    }
  }

  $: filteredClaims = selectedClaimCountry
    ? claimsFeatures.filter((cl) => {
        for (let i = 1; i <= 8; i++) {
          const key = `SOVEREIGN${i}`;
          if (cl.properties?.[key] === selectedClaimCountry) return true;
        }
        return false;
      })
    : claimsFeatures;

  let highlightApprovedProposals = false;

  function handleToggleApprovedProposals(active) {
    highlightApprovedProposals = active;
  }

  window.addEventListener("DOMContentLoaded", (event) => {
    function updateIframeHeight() {
      const el = document.documentElement;
      const rect = el.getBoundingClientRect();
      const styles = window.getComputedStyle(el);
      const margin =
        parseFloat(styles.marginTop) + parseFloat(styles.marginBottom);
      const height = Math.ceil(rect.height + margin);

      window.parent.postMessage(
        {
          type: "resize-iframe",
          value: height,
        },
        "*",
      );
    }
    updateIframeHeight();

    if (window.ResizeObserver) {
      new ResizeObserver(() => {
        updateIframeHeight();
      }).observe(document.documentElement);
    } else {
      window.addEventListener("load", updateIframeHeight);
      window.addEventListener("resize", updateIframeHeight);
    }

    window.addEventListener(
      "message",
      (event) => {
        if (event.data.type === "request-resize") {
          updateIframeHeight();
        }
      },
      false,
    );
  });
</script>

<div class="chart-container" bind:clientWidth={width}>
  <div class="legend">
    <div class="legend-item">
      <span class="legend-color zee-color"></span>
      ZEE
    </div>
    <div class="legend-item">
      <span class="legend-color claims-color"></span>
      Reclamaciones
    </div>
    <ToggleButton
      active={highlightApprovedProposals}
      onToggle={handleToggleApprovedProposals}
      label="Ver aprobadas"
      activeColor="#2e6ecf"
    />
    <ToggleButton active={highlightECS} onToggle={handleToggle} />
  </div>

  <ClaimsCountrySelector
    claims={claimsFeatures}
    selectedCountry={selectedClaimCountry}
    onSelect={handleCountrySelect}
  />
  <svg {width} {height} bind:this={globe} class:dragging>
    <Glow />

    <circle
      cx={width / 2}
      cy={height / 2}
      r={width / 2}
      fill="#cadee7"
      filter="url(#glow)"
      on:click={() => (tooltipData = null)}
    />

    {#each claimsFeatures as cl, i (i)}
      <path
        class="claims"
        d={path(cl)}
        fill={selectedClaimCountry &&
        [...Array(8).keys()].some(
          (i) => cl.properties?.[`SOVEREIGN${i + 1}`] === selectedClaimCountry,
        )
          ? "#0060f0"
          : highlightApprovedProposals &&
              (cl.properties?.POL_TYPE ===
                "Propuesta aprobada total o parcialmente" ||
                cl.properties?.POL_TYPE ===
                  "Propuesta conjunta aprobada total o parcialmente")
            ? "#2e6ecf"
            : "#76a7f0"}
        stroke="white"
        stroke-width="0.2"
        on:click={(event) => handleClaimClick(cl, event)}
      />
    {/each}
    {#if highlightECS}
      {#each claimsFeatures.filter((cl) => cl.properties?.POL_TYPE === "Propuesta solapada") as cl}
        <path
          d={path(cl)}
          fill="none"
          stroke="#ff4500"
          stroke-width="1"
          pointer-events="none"
        />
      {/each}
    {/if}

    {#each zeeFeatures as zee}
      <path d={path(zee)} fill="#ebf3ff" stroke="none" />
    {/each}

    {#each countries as c}
      <path
        d={path(c)}
        fill={countryToISO[selectedClaimCountry] === String(c.id) // o c.properties.id
          ? "#555555"
          : "#cccccc"}
        stroke="#cccccc"
      />
    {/each}

    <circle
      cx={width / 2}
      cy={height / 2}
      r={width / 2}
      fill="none"
      stroke="black"
      stroke-width="0.4px"
    />

    {#if tooltipData}
      {#key tooltipData.id}
        <path
          d={path(tooltipData)}
          fill="none"
          stroke="white"
          stroke-width="2"
        />
      {/key}
    {/if}
  </svg>

  <Tooltip data={tooltipData} />
</div>

<style>
  :global(body) {
    background-color: #cadee7;
  }
  .chart-container {
    max-width: 900px;
    margin: 0 auto;
  }
  svg {
    overflow: visible;
  }
  .dragging {
    cursor: grabbing;
  }
  .claims {
    cursor: pointer;
  }
  .legend {
    display: flex;
    flex-wrap: wrap; /* permite que los items se bajen a otra línea */
    gap: 1rem; /* reduce un poco el gap para móviles */
    justify-content: center;
    user-select: none;
    margin-bottom: 0.7rem;
    font-size: 0.9rem;
  }
  .legend-item {
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }
  .legend-color {
    width: 15px;
    height: 15px;
    border-radius: 50%;
    display: inline-block;
  }
  .zee-color {
    background-color: #ebf3ff;
  }
  .claims-color {
    background-color: #76a7f0;
  }
  .approved-proposal-color {
    background-color: #28a745; /* verde tipo "aprobado" */
  }
</style>
