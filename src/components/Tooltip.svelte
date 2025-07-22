<script>
  export let data;

  // Extrae todos los territorios no nulos de las propiedades
  function getTerritories(properties) {
    const territories = [];
    for (let i = 1; i <= 8; i++) {
      const key = `SOVEREIGN${i}`;
      if (properties[key]) {
        territories.push(properties[key]);
      }
    }
    return territories;
  }

  $: territories = data ? getTerritories(data.properties) : [];
</script>

{#if data}
  <div class="tooltip">
    {#if territories.length > 0}
      <p><strong>{territories.join(", ")}</strong></p>
    {/if}
    <p>Tipo: {data.properties?.POL_TYPE?.toLowerCase()}</p>
  </div>
{/if}

<style>
  div {
    position: absolute;
    bottom: 30px;
    right: 0;
    text-align: right;
  }

  .tooltip {
    background-color: rgba(0, 0, 0, 0.75);
    color: white;
    padding: 0.6rem 1rem;
    border-radius: 0.4rem;
    max-width: 350px;
    font-size: 0.9rem;
    box-shadow: 0 0 8px rgba(0, 0, 0, 0.7);
  }

  strong {
    display: block;
    margin-bottom: 0.3rem;
    font-size: 1.1rem;
  }

  p {
    margin: 0.3rem 0;
  }
</style>
