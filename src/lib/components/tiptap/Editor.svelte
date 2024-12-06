<script lang="ts">
	import { onDestroy, onMount, unmount } from 'svelte';
	import { Editor, Extension } from '@tiptap/core';
	import StarterKit from '@tiptap/starter-kit';
	import Suggestion from '@tiptap/suggestion';
	import CommandsList from './CommandList.svelte';
	import tippy, { type Instance } from 'tippy.js';
	import { mount } from 'svelte';
	import CommandList from './CommandList.svelte';

	let element: HTMLDivElement;
	let commandsListEl: HTMLDivElement | undefined = $state();
	let editor: Editor | undefined = $state();

	let commandsListProps = $state({ props: { items: [] } });

	type CommandType = {
		editor: any;
		range: any;
		props: any;
	};

	const commands = Extension.create({
		name: 'commands',

		addOptions() {
			return {
				suggestion: {
					char: '/',
					command: ({ editor, range, props }: CommandType) => {
						props.command({ editor, range });
					}
				}
			};
		},

		addProseMirrorPlugins() {
			return [
				Suggestion({
					editor: this.editor,
					...this.options.suggestion
				})
			];
		}
	});

	const suggestion = {
		items: ({ query }: { query: any }) => {
			return [
				{
					title: 'Heading 1',
					command: ({ editor, range }: CommandType) => {
						editor.chain().focus().deleteRange(range).setNode('heading', { level: 1 }).run();
					}
				},
				{
					title: 'Heading 2',
					command: ({ editor, range }: CommandType) => {
						editor.chain().focus().deleteRange(range).setNode('heading', { level: 2 }).run();
					}
				},
				{
					title: 'Bold',
					command: ({ editor, range }: CommandType) => {
						editor.chain().focus().deleteRange(range).setMark('bold').run();
					}
				},
				{
					title: 'Italic',
					command: ({ editor, range }: CommandType) => {
						editor.chain().focus().deleteRange(range).setMark('italic').run();
					}
				}
			]
				.filter((item) => item.title.toLowerCase().startsWith(query.toLowerCase()))
				.slice(0, 10);
		},

		render: () => {
			let component: CommandList;
			let popup: Instance[];

			return {
				onStart: (props: any) => {
					commandsListProps.props = props;
					component = mount(CommandsList, {
						target: commandsListEl!,
						props: commandsListProps.props as any
					});

					if (!props.clientRect) {
						return;
					}

					popup = tippy('body', {
						getReferenceClientRect: props.clientRect,
						appendTo: () => document.body,
						content: commandsListEl,
						showOnCreate: true,
						interactive: true,
						trigger: 'manual',
						placement: 'bottom-start'
					});
				},

				onUpdate(props: any) {
					console.log('updae', props);
					commandsListProps.props.items = props.items;

					if (!props.clientRect) {
						return;
					}

					popup[0].setProps({
						getReferenceClientRect: props.clientRect
					});
				},

				onKeyDown(props: any) {
					if (props.event.key === 'Escape') {
						popup[0].hide();

						return true;
					}

					return component.onKeyDown(props.event);
				},

				onExit() {
					if (popup.length) {
						popup[0].destroy();
					}
					unmount(component);
				}
			};
		}
	};

	onMount(() => {
		editor = new Editor({
			element: element,
			extensions: [
				StarterKit,
				commands.configure({
					suggestion
				})
			],
			content: '<p>Hello World! 🌍️ </p>',
			onTransaction: () => {
				// force re-render so `editor.isActive` works as expected
				editor = editor;
			}
		});
	});

	onDestroy(() => {
		if (editor) {
			editor.destroy();
		}
	});
</script>

<div bind:this={element}></div>
{#if editor}
	<div bind:this={commandsListEl}></div>
	<p class="mt-2">
		Type <span>/</span> to see columns
	</p>
{/if}

<style lang="postcss">
	:global(.tiptap) {
		@apply rounded-lg border px-3 py-4;
	}
</style>
