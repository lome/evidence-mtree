<script>
	import { writable } from 'svelte/store';
	import { onMount } from 'svelte';
	export const clear = writable(false);

	export let name = "";
	export let children = [];
	export let filter = null;
	export let addFilter;
	export let removeFilter;
	export let markSelected;
	export let clearWritable;
	
	let open = false;
	let selected = false;
	let selectedChildren = {};
	let hasChildSelected = false;

	$: selected
	
	$: {
		// Add invoke function to children
		if (children){
			children.forEach(child => {
				child.markSelected = markChildSelected;
				child.clearWritable = clear;
			});
		}
	}

	onMount(() => {
		if (clearWritable) {
			clearWritable.subscribe(value => {
				console.log('Cleared!',value);
				//Clear node
				selected = false;
				removeFilter(filter);
			});
		}
   });

	function markChildSelected(_selected, key){
		if (_selected) selectedChildren[key] = true;
		else selectedChildren[key] = false;
		hasChildSelected = Object.keys(selectedChildren)
			.filter(k => selectedChildren[k] == true).length > 0;
		if (hasChildSelected){
			selected = false;
			refreshData();
		}
	}

	function toggleOpen() {
		open = hasChildSelected || !open;
	}

	function toggleSelected() {
		// Main node, break
		if (Object.keys(filter).length == 0) return;

		selected = !selected;
		if (markSelected){
			markSelected(selected, ''+filter._key);
		}
		refreshData();
	}

	function refreshData(){
		if (selected){
			//console.log('Selected',  filter);
			addFilter(filter);
			clear.set(Date.now());
		} else {
			//console.log('UnSelected', filter);
			removeFilter(filter);
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
	{(selected) ? 'font-bold' : ''}" 
		on:click={toggleSelected}>{name}</span>
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


