<script lang="ts">
	type Props = {
		items: any[];
		open: boolean;
		command: (index: number) => void;
	};
	let { items, command, open = $bindable(false) }: Props = $props();

	let selectedIndex: number = $state(0);

	$effect(() => {
		if (items) {
			selectedIndex = 0;
		}
	});

	export function onKeyDown(event: KeyboardEvent) {
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
			open = false;
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

	export function enterHandler() {
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
		<div class="item">No columns to select</div>
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
