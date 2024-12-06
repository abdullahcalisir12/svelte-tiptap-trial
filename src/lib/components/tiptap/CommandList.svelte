<script lang="ts">
	type Props = {
		items: any[];
		command: (index: number) => void;
	};
	let { items, command }: Props = $props();

	let selectedIndex: number = $state(0);

	$effect(() => {
		console.log('items', items);
		if (items) {
			selectedIndex = 0;
		}
	});

	export function onKeyDown(event: KeyboardEvent) {
		console.log(event);
		if (event.key === 'ArrowUp') {
			upHandler();
			return true;
		}

		if (event.key === 'ArrowDown') {
			downHandler();
			return true;
		}

		if (event.key === 'Enter') {
			enterHandler();
			return true;
		}

		return false;
	}

	function upHandler() {
		selectedIndex = (selectedIndex + items.length - 1) % items.length;
	}

	function downHandler() {
		selectedIndex = (selectedIndex + 1) % items.length;
	}

	function enterHandler() {
		selectItem(selectedIndex);
	}

	function selectItem(index: number) {
		const item = items[index];

		if (item) {
			command(item);
		}
	}
</script>

<div class="dropdown-menu">
	{#if items.length}
		{#each items as item, i (i)}
			<button class:is-selected={i === selectedIndex} onclick={() => selectItem(i)}>
				{item.title}
			</button>
		{/each}
	{:else}
		<div class="item">No result</div>
	{/if}
</div>

<style lang="postcss">
	/* Dropdown menu */
	.dropdown-menu {
		@apply relative flex flex-col gap-1 overflow-auto rounded-lg border bg-white p-2 shadow;

		button {
			@apply flex w-full items-center gap-1 rounded bg-transparent px-2 py-1 text-left;

			&:hover,
			&:hover.is-selected {
				@apply bg-gray-300;
			}

			&.is-selected {
				@apply bg-gray-200;
			}
		}
	}
</style>
