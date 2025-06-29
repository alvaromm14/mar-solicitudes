<script>
  import { scaleBand, scaleSqrt } from "d3-scale";
  import { arc } from "d3-shape";
  import { max } from "d3-array";
  import rawData from "$data/data.js";
  import Tooltip from "./components/Tooltip.svelte";

  let data = rawData.map((d) => ({ ...d })).sort((a, b) => b.new - a.new);

  const margin = { top: 55, right: 0, bottom: 0, left: 0 };
  let width = 1000;
  $: height = width * 0.75;
  $: innerWidth = width - margin.left - margin.right;
  $: innerHeight = height - margin.top - margin.bottom;
  $: radiusX = innerWidth / 2;
  $: radiusY = innerHeight / 2;
  $: baseInnerRadius = 125;

  $: angleScale = scaleBand()
    .domain(data.map((d) => d.country))
    .range([0, 2 * Math.PI])
    .padding(0.05);

  $: radialScale = scaleSqrt()
    .domain([0, max(data, (d) => d.new)])
    .range([0, Math.min(radiusX, radiusY) - baseInnerRadius]);

  $: arcData = data.map((d) => {
    const startAngle = angleScale(d.country);
    const endAngle = startAngle + angleScale.bandwidth();
    const angle = (startAngle + endAngle) / 2;
    const r0 = baseInnerRadius;
    const r1 = baseInnerRadius + radialScale(d.new);

    const labelRadius = r1 + 5;
    const labelX = Math.cos(angle - Math.PI / 2) * labelRadius;
    const labelY = Math.sin(angle - Math.PI / 2) * labelRadius;

    return {
      country: d.country,
      new: d.new,
      increase: d.increase,
      path: arc()({
        innerRadius: r0,
        outerRadius: r1,
        startAngle,
        endAngle,
      }),
      gradientId: `grad-${d.country.replace(/\s+/g, "")}`,
      angle,
      labelX,
      labelY,
    };
  });

  let hovered = null;

  function handleMouseOver(d) {
    hovered = d;
  }

  function handleMouseOut() {
    hovered = null;
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

<div
  class="chart-container"
  bind:clientWidth={width}
  bind:clientHeight={height}
  style="position: relative; height: 100vh;"
>
  <h2>Extensión de las ampliaciones solicitadas por cada país</h2>
  <svg {width} {height}>
    <defs>
      {#each arcData as d}
        <radialGradient id={d.gradientId} cx="50%" cy="50%" r="70%">
          <stop offset="0%" stop-color="#c7ddef" />
          <stop offset="50%" stop-color="#7aaac7" />
          <stop offset="100%" stop-color="#3c6e94" />
        </radialGradient>
      {/each}
    </defs>

    <g
      transform={`translate(${margin.left + innerWidth / 2}, ${margin.top + innerHeight / 2})`}
    >
      {#each arcData as d}
        <path
          d={d.path}
          fill={`url(#${d.gradientId})`}
          stroke="white"
          stroke-width="0.5"
          opacity={hovered ? (hovered.country === d.country ? 1 : 0.5) : 1}
          on:mouseover={() => handleMouseOver(d)}
          on:mouseout={handleMouseOut}
        />
      {/each}
      {#if hovered}
        <Tooltip
          country={hovered.country}
          total={hovered.new}
          increase={hovered.increase}
          visible={true}
        />
      {/if}
      {#each arcData as d}
        <text
          x={d.labelX}
          y={d.labelY}
          font-size={hovered?.country === d.country ? "0.75rem" : "0.65rem"}
          font-weight={d.country === "España" && !hovered
            ? "bold"
            : hovered?.country === d.country
              ? "bold"
              : "normal"}
          text-anchor={d.angle > Math.PI ? "end" : "start"}
          alignment-baseline="middle"
          transform={`rotate(${(d.angle * 180) / Math.PI - 90 > 90 ? (d.angle * 180) / Math.PI + 90 : (d.angle * 180) / Math.PI - 90}, ${d.labelX}, ${d.labelY})`}
          pointer-events="none"
          opacity={!hovered || hovered.country === d.country ? 1 : 0}
          style="transition: opacity 0.3s ease, font-size 0.3s ease;"
        >
          {d.country}
        </text>
      {/each}
    </g>
  </svg>
</div>

<style>
  :global(body) {
    background-color: #cadee7;
  }
  .chart-container {
    max-width: 900px;
    margin: 0 auto;
  }
  h2 {
    text-align: center;
  }
  h2 {
    font-size: 1.25rem;
    font-weight: 400;
    margin-bottom: 0.5rem;
  }
  .chart-container {
    margin: 0 auto;
  }

  path:hover {
    opacity: 0.8;
    cursor: pointer;
  }

  path {
    transition: opacity 0.3s ease;
  }
</style>
