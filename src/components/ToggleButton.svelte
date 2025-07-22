<script>
  export let active = false;
  export let onToggle = () => {};
  export let label = "Ver solapamientos";
  export let activeColor = "#ff4500";

  function toggle() {
    onToggle(!active);
  }
</script>

<div class="toggle-wrapper" on:click={toggle}>
  <div class="toggle-track">
    <div class="toggle-knob" class:active={active} style={active ? `background-color: ${activeColor}` : ""}></div>
  </div>
    <span class="label">{label}</span>
</div>

<style>
  .toggle-wrapper {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    cursor: pointer;
    user-select: none;

  }

  .toggle-track {
    width: 40px;
    height: 19px;
    background-color: #555;
    border-radius: 9999px;
    position: relative;
    transition: background-color 0.3s ease;
  }

  .toggle-knob {
    width: 15px;
    height: 15px;
    background-color: white;
    border-radius: 50%;
    position: absolute;
    top: 2px;
    left: 2px;
    transition: left 0.3s ease, background-color 0.3s ease;
    box-shadow: 0 0 2px rgba(0, 0, 0, 0.3);
  }

  .toggle-knob.active {
    left: 23px;
    background-color: #ff4500;
  }
</style>
