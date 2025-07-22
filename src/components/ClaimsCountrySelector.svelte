<script>
  export let claims = [];
  export let selectedCountry = "";
  export let onSelect = (country) => {};

  // Contar cuántos claims tiene cada territorio
  $: territoryCounts = claims.reduce((acc, claim) => {
    const props = claim.properties || {};
    for (let i = 1; i <= 8; i++) {
      const key = `SOVEREIGN${i}`;
      if (props[key]) {
        acc[props[key]] = (acc[props[key]] || 0) + 1;
      }
    }
    return acc;
  }, {});

  // Crear array con territorios únicos y sus cuentas, ordenado de mayor a menor
  $: allTerritories = Object.entries(territoryCounts)
    .sort((a, b) => b[1] - a[1])
    .map(([territory, count]) => ({ territory, count }));

  function handleChange(event) {
    onSelect(event.target.value);
  }
</script>

<div class="selector">
  <label for="territory">Elige un país:</label>
  <select id="territory" bind:value={selectedCountry} on:change={handleChange}>
    <option value="">-- Todos --</option>
    {#each allTerritories as { territory, count }}
      <option value={territory}>{territory} ({count})</option>
    {/each}
  </select>
</div>

<style>
  .selector {
    margin-bottom: 1rem;
    font-size: 0.9rem;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 0.9rem;
  }

  select {
    margin-left: 0.3rem;
    padding: 0.2rem 0.3rem;

    border-radius: 0.3rem;
    border: 1px solid #ccc;
  }
</style>
