<script lang="ts">
	import { onDestroy, onMount, unmount } from 'svelte';
	import { Editor, Extension, Node, mergeAttributes } from '@tiptap/core';
	import StarterKit from '@tiptap/starter-kit';
	import Suggestion from '@tiptap/suggestion';
	import CommandsList from './CommandList.svelte';
	import tippy, { type Instance } from 'tippy.js';
	import { mount } from 'svelte';
	import CommandList from './CommandList.svelte';

	let { columns }: { columns: string[] } = $props();

	let element: HTMLDivElement;
	let commandsListEl: HTMLDivElement | undefined = $state();
	let editor: Editor | undefined = $state();

	let commandsListProps = $state({ props: { items: [], open: false } });
	let component: CommandList;

	type CommandType = {
		editor: Editor;
		range: any;
		props: any;
	};

	const commands = Extension.create({
		priority: 1,
		name: 'slash-commands',

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

	const ColumnNode = Node.create({
		name: 'nodeView',
		group: 'inline',
		inline: true,
		atom: true,

		addAttributes() {
			return {
				text: {
					default: ''
				}
			};
		},

		parseHTML() {
			return [
				{
					tag: 'node-view'
				}
			];
		},

		renderHTML({ HTMLAttributes }) {
			return ['node-view', mergeAttributes(HTMLAttributes)];
		},

		addNodeView() {
			return ({ node }) => {
				const dom = document.createElement('span');

				dom.classList.add('ob-column-title');

				dom.innerHTML = node.attrs.text;

				return {
					dom
				};
			};
		}
	});

	const suggestion = {
		items: ({ query }: { query: any }) => {
			return [
				...columns.map((c) => {
					return {
						title: c,
						command: ({ editor, range }: CommandType) => {
							try {
								editor
									.chain()
									.focus()
									.deleteRange(range)
									.insertContent([
										{
											type: 'nodeView',
											attrs: {
												text: c
											}
										},
										{
											type: 'text',
											text: ' ',
											isInline: true
										}
									])
									.run();
							} catch (error) {
								console.error(error);
							}
						}
					};
				})
			]
				.filter((item) => item.title.toLowerCase().startsWith(query.toLowerCase()))
				.slice(0, 10);
		},

		render: () => {
			let popup: Instance[];

			return {
				onStart: (props: any) => {
					commandsListProps.props = props;
					commandsListProps.props.open = true;
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
			editorProps: {
				handleDOMEvents: {
					keydown: (_, event) => {
						if (event.key === 'Enter' && commandsListProps.props.open) {
							event.preventDefault();
							component.enterHandler();
							commandsListProps.props.open = false;
						}
					}
				}
			},
			element: element,
			extensions: [
				StarterKit,
				ColumnNode,
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

<div class="relative flex h-full flex-col gap-1">
	<div bind:this={element} class="flex-1"></div>
	{#if editor}
		<div bind:this={commandsListEl}></div>
		<p class="flex w-full items-center rounded-lg bg-gray-100 px-2 py-1 text-sm">
			Type <span class="mx-2 rounded bg-blue-500 text-white">
				<svg
					xmlns="http://www.w3.org/2000/svg"
					fill="none"
					viewBox="0 0 24 24"
					stroke-width={1.5}
					stroke="currentColor"
					class="size-5"
				>
					<path stroke-linecap="round" stroke-linejoin="round" d="m9 20.247 6-16.5" />
				</svg>
			</span> to see columns
		</p>
	{/if}
</div>

<style lang="postcss">
	:global(.tiptap) {
		@apply relative min-h-full rounded-lg border px-4 py-3;
	}

	:global(.ob-column-title) {
		@apply !whitespace-nowrap rounded-full border bg-blue-500 px-2 py-1 text-xs text-white;
	}
</style>
