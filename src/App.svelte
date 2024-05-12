<script lang="ts">
	import Setting from "./assets/Setting.svelte";
	import Randomizing from "./assets/Randomizing.svelte";

	let col = Number(localStorage.getItem("col") || 6);
	let row = Number(localStorage.getItem("row") || 5);
	let disabled_seats: string[] = JSON.parse(localStorage.getItem("disabled_seats") || "[]");
	let is_started_randomizing = false;
</script>

<svelte:head>
  	<title>무작위 학급 자리 배치</title>
</svelte:head>

<main>
	{#if !is_started_randomizing}
		<Setting bind:col={col} bind:row={row} bind:disabled_seats={disabled_seats} bind:is_started_randomizing={is_started_randomizing}></Setting>
	{:else}
		<Randomizing col={col} row={row} disabled_seats={disabled_seats} bind:is_started_randomizing={is_started_randomizing}></Randomizing>
	{/if}
</main>

<style>
	main {
		display: flex;
		flex-direction: column;
		align-items: center;
		padding-top: 20px;
	}
</style>
