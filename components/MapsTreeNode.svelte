<script>
	import { writable } from 'svelte/store';
	import { onMount } from 'svelte';
	export const clearChildren = writable(false);

	export let name = "";
	export let children = [];
	export let filter = null;
	export let addFilter;
	export let removeFilter;
	export let markSelected;
	export let parentSelected;
	
	let open = false;
	let selected = false;
	let selectedChildren = {};

	$: selected
	
	$: {
		// Add invoke function to children
		if (children){
			children.forEach(child => {
				child.markSelected = markChildSelected;
				child.parentSelected = clearChildren;
			});
		}
	}

	onMount(() => {
		if (parentSelected) {
			parentSelected.subscribe(value => {
				//Clear node
				selected = false;
				refreshData();
			});
		}
   });

	function markChildSelected(_selected, key){
		selectedChildren[key] = _selected;
		if (selected && hasChildSelected()){
			selected = false;
			refreshData();
		}
	}

	function hasChildSelected(){
		return Object.values(selectedChildren)
			.filter(k => k == true).length > 0;
	}

	function toggleOpen() {
		//console.log('children',selectedChildren);
		open = hasChildSelected() || !open;
	}

	function toggleSelected() {
		// Main node, break
		if (Object.keys(filter).length == 0) return;
		selected = !selected;
		refreshData();
	}

	function refreshData(){
		if (selected){
			//console.log('Selected',  filter);
			addFilter(filter);
			clearChildren.set(Date.now());
			if (markSelected){
				markSelected(selected, ''+filter._key);
			}
		} else {
			//console.log('UnSelected', filter);
			removeFilter(filter);
			if (markSelected){
				markSelected(selected, ''+filter._key);
			}
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


