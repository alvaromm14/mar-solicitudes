<script>
  import { scaleLinear } from "d3-scale";
  export let country = "";
  export let total = 0;
  export let increase = 0;
  export let visible = false;

    const colorScale = scaleLinear()
    .domain([0, 2.6])
    .range(["#d0e7f9", "#145374"])
    .clamp(true);
    
  $: increaseText = `+${Math.round(increase * 100).toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".")}%`;

  const rectHeight = 20;
  const rectPaddingX = 6;
  const approxCharWidth = 8; // ancho aproximado por caracter en px

  $: bgColor = colorScale(increase);
    $: percentage = Math.round(increase * 100);
  $: textColor = percentage <= 80 ? "#000" : "#fff";
</script>

{#if visible}
  <text
    text-anchor="middle"
    alignment-baseline="middle"
    fill="#222"
    font-weight="bold"
    font-size="18"
  >
    <tspan x="0" font-weight="normal" font-size="14">
      {total.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".")} km²
    </tspan>
  </text>

  <g transform="translate(0, 18)">
    <rect
      x={-increaseText.length * approxCharWidth / 2 - rectPaddingX}
      y={-rectHeight / 2}
      width={increaseText.length * approxCharWidth + rectPaddingX * 2}
      height={rectHeight}
        fill={bgColor}
      rx="4"
      ry="4"
    />
    <text
      x="0"
      y="1"
      text-anchor="middle"
      alignment-baseline="middle"
      font-weight="normal"
      font-size="14"
        fill={textColor}
      pointer-events="none"
    >
      {increaseText}
    </text>
  </g>
{/if}
