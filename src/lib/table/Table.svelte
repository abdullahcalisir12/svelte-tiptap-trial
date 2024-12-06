<script lang="ts">
	import { preventDefault, stopPropagation } from 'svelte/legacy';

	let { columns = $bindable([]) }: { columns: string[] } = $props();

	let newColumnValue = $state('');
</script>

<div class="flex flex-col gap-1 rounded-lg bg-gray-100 px-3 py-4">
	<h3 class="-mt-1.5 mb-1.5 font-bold">Table Columns</h3>
	{#each columns as c, i}
		<div class="flex items-center gap-2">
			<input type="text" class="flex-1 rounded" bind:value={columns[i]} />
			<button
				class="grid aspect-square place-content-center rounded-full p-1 text-rose-500 hover:bg-rose-400 hover:text-white"
				aria-label="Remove Column"
				onclick={() => {
					columns.splice(i, 1);
				}}
			>
				<svg
					xmlns="http://www.w3.org/2000/svg"
					fill="none"
					viewBox="0 0 24 24"
					stroke-width="1.5"
					stroke="currentColor"
					class="size-5"
				>
					<path stroke-linecap="round" stroke-linejoin="round" d="M6 18 18 6M6 6l12 12" />
				</svg>
			</button>
		</div>
	{/each}
	<form
		class="flex items-center gap-2"
		onsubmit={(e) => {
			e.preventDefault();
			e.stopPropagation();
			if (newColumnValue) {
				columns.push(newColumnValue);
				newColumnValue = '';
			}
		}}
	>
		<input
			type="text"
			name="column-name"
			required
			class="flex-1 rounded"
			placeholder="New Column"
			bind:value={newColumnValue}
			autocomplete="off"
		/>
		<button
			class="grid aspect-square place-content-center rounded-full p-1 hover:bg-green-100"
			aria-label="Remove Column"
			type="submit"
		>
			<svg
				xmlns="http://www.w3.org/2000/svg"
				fill="none"
				viewBox="0 0 24 24"
				stroke-width="1.5"
				stroke="currentColor"
				class="size-5"
			>
				<path stroke-linecap="round" stroke-linejoin="round" d="M12 4.5v15m7.5-7.5h-15" />
			</svg>
		</button>
	</form>
</div>
