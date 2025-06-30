<script>
	import { getInputContext } from '@evidence-dev/sdk/utils/svelte';
	export let name = "";
	export let children = [];
	export let filter = null;
	export let input_name = null;
	const inputs = getInputContext();
	
	let open = false;
	let selected = false;

	$: selected
	$: {
		if (filter){
			let s = true;
			filter.forEach(f => {
				Object.keys(f).forEach(k => {
					if ($inputs[input_name] && $inputs[input_name][k]){
						s = s && $inputs[input_name][k] == f[k];
					}
				});
			});
			selected = s;
			console.log(selected);
		}
	}
	
	function toggleOpen() {
		open = !open;
		if (children.length == 0){
			let value = {};
			filter.forEach(f => {
				Object.keys(f).forEach(k => {
					value[k] = f[k];
					//console.log(k,'=>',f[k]);
				});
			});
			$inputs[input_name] = value;
			//console.log(input_name,$inputs);
		}
	}
</script>

<div class="tree mb-0 mt-0">
  <div class="tree-node items-center py-1">
	{#if children.length}
		{#if open}
		<button class="expand-button mr-2 text-sm text-base-content cursor-pointer inline-flex gap-2" 
		on:click={toggleOpen}>-</button>
		{:else}
		<button class="expand-button mr-2 text-sm text-base-content cursor-pointer inline-flex gap-2" 
		on:click={toggleOpen}>+</button>
		{/if}
	{:else}
		<button class="expand-button mr-2 text-sm text-base-content cursor-pointer inline-flex gap-2">&nbsp</button>
	{/if}
    <span class="node-label text-sm text-base-content cursor-pointer inline-flex gap-2 
	{selected ? ( children.length ? 'font-semibold' : 'font-bold' ) : ''}" 
		on:click={toggleOpen}>{name}</span>
	{#if open}
    <div class="children ml-4">
      {#each children as child}
			<svelte:self {...child} />
		{/each}
    </div>
	{/if}
  </div>
</div>


<style>

</style>


