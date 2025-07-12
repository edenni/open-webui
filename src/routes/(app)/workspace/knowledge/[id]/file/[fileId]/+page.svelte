<script lang="ts">
	import { page } from '$app/stores';
	import { getFileChunksById } from '$lib/apis/files';
	import { onMount } from 'svelte';
	import { get, writable } from 'svelte/store';

	const chunks = writable<any[]>([]);
	const loading = writable(true);

	onMount(async () => {
		const params = get(page).params as { id: string; fileId: string };
		try {
			const data = await getFileChunksById(localStorage.token, params.fileId);
			chunks.set(data ?? []);
		} catch (e) {
			console.error(e);
		} finally {
			loading.set(false);
		}
	});
</script>

<svelte:head>
	<title>File Chunks</title>
</svelte:head>

{#if $loading}
	<div class="flex w-full h-full">
		<div class="m-auto text-sm text-gray-500">Loading...</div>
	</div>
{:else}
	<div class="flex flex-col w-full p-4 space-y-2">
		<h1 class="text-xl font-semibold">文件分块</h1>
		{#if $chunks.length === 0}
			<div class="text-sm text-gray-500">暂无分块数据</div>
		{:else}
			<div class="space-y-2">
				{#each $chunks as chunk}
					<div class="border border-gray-100 dark:border-gray-800 rounded-lg p-3">
						<div class="text-xs text-gray-400 mb-1">Chunk {chunk.index}</div>
						<pre class="whitespace-pre-wrap text-sm">{chunk.content}</pre>
					</div>
				{/each}
			</div>
		{/if}
	</div>
{/if} 