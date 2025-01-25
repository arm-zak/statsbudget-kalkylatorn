<script>
    import { Chart as ChartJS, Title, Tooltip, Legend, ArcElement } from "chart.js";
    import PieChart from './pichart.svelte';
    import utgifter from './utgifter.json';
    import { writable, derived } from 'svelte/store';

    ChartJS.register(Title, Tooltip, Legend, ArcElement);
    
    let newName = "";
    let newValue = "";
    const budget = writable(utgifter);

    const expanded = writable({});

    function toggleExpand(index) {
      expanded.update(exp => {
        exp[index] = !exp[index];
        return exp;
      });
    }

    function updatePieChart() {
  // This function is called whenever an input value changes
  // No need to do anything here as Svelte will reactively update the data
  }

  function formatNumber(value) {
    return value.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',');
  }

  function addRow() {
    budget.update(b => {
      const key = `category${Object.keys(b).length + 1}`;
      b[key] = { name: newName, value: parseFloat(newValue) * 1000, items: {} };
      newName = "";
      newValue = "";
      return b;
    });
  }

  const total = derived(budget, $budget => {
    return Object.values($budget).reduce((total, { value }) => total + value, 0);
  });

  $: {
    budget.update(b => {
      Object.entries(b).forEach(([key, { items }]) => {
        b[key].value = Object.values(items).reduce((sum, { value }) => sum + value, 0);
      });
      return b;
    });
  }
</script>

<h2>Statens utgifter: {formatNumber($total)} SEK</h2>
<div class="container">
  <table>
    <thead>
      <tr>
        <th>Område</th>
        <th>Utgift</th>
      </tr>
    </thead>
    <tbody>
      {#each Object.entries($budget) as [key, { name, value, items }]}
        <tr>
          <td on:click={() => toggleExpand(key)}>{name}</td>
          <td>
            <input type="number" bind:value={$budget[key].value} on:input={updatePieChart} style="text-align: right;">
          </td>
        </tr>
        {#if $expanded[key]}
          {#each Object.entries(items) as [subKey, { name, value }]}
            <tr>
              <td style="padding-left: 20px;">{name}</td>
              <td>
                <input type="number" bind:value={$budget[key].items[subKey].value} on:input={updatePieChart} style="text-align: right;">
              </td>
            </tr>
          {/each}
        {/if}
      {/each}
      <tr>
        <td colspan="2">
          <form on:submit|preventDefault={addRow} style="display: flex; justify-content: space-between;">
            <input type="text" bind:value={newName} placeholder="Name" required style="flex: 0; margin-right: 100px;">
            <input type="number" bind:value={newValue} placeholder="Value" required style="flex: 0; margin-right: 100px;">
            <button type="submit" style="flex: 1; font-weight: bold;">Lägg till</button>
          </form>
        </td>
      </tr>
    </tbody>
  </table>

  <PieChart data={$budget} />
</div>

<style>
  .container {
    display: flex;
    align-items: flex-start;
  }
  table {
    width: 50%;
  }
  th, td {
    text-align: right;
    padding: 2px;
  }
  th:first-child, td:first-child {
    text-align: left;
  }
  h2 {
    font-size: 2em; /* Adjust the size as needed */
  }
  form {
    width: 100%;
  }
  input {
    padding: 5px;
  }
  button {
    padding: 5px 10px;
  }
</style>