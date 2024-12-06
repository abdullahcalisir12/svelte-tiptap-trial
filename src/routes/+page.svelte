<script>
	import Editor from '$lib/components/tiptap/Editor.svelte';
	import Table from '$lib/table/Table.svelte';

	let columns = $state(['Firstname', 'Lastname', 'Linkedin URI', 'Company']);
</script>

<div class="flex min-h-screen w-screen flex-col items-center justify-center">
	<main
		class="container mx-auto grid grid-cols-1 place-content-center gap-3 p-4 md:grid-cols-[396px_396px]"
	>
		<Table bind:columns />
		<Editor {columns} />
	</main>
</div>
