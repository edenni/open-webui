<script lang="ts">
	import GarbageBin from '$lib/components/icons/GarbageBin.svelte';
	import { formatFileSize } from '$lib/utils';
	import dayjs from 'dayjs';
	import { createEventDispatcher } from 'svelte';

	const formatTimestamp = (timestamp: number) => {
		if (!timestamp) return '-';
		return dayjs.unix(timestamp).format('YYYY-MM-DD HH:mm');
	};

	const dispatch = createEventDispatcher();

	export let selectedFileId: string | null = null;
	export let files: any[] = [];

	export let small = false;
</script>

<!-- 文件行 -->
<div class="max-h-full flex flex-col w-full divide-y divide-gray-50 dark:divide-gray-850">
	{#each files as file}
		<div
			class="w-full px-2 py-1 text-xs hover:bg-gray-50 dark:hover:bg-gray-850 grid gap-2 items-center {selectedFileId === file.id ? 'bg-gray-50 dark:bg-gray-850' : ''}"
			style="grid-template-columns:2fr 1.2fr 1fr auto;"
		>
			<!-- 文件信息点击区域 -->
			<button
				type="button"
				class="col-span-3 grid grid-cols-3 text-left items-center gap-2 w-full"
				style="grid-template-columns:2fr 1.2fr 1fr;"
				on:click={() => dispatch('click', file.id)}
			>
				<div class="truncate">{file?.meta?.name ?? file?.name}</div>
				<div>{formatTimestamp(file.created_at)}</div>
				<div class="text-right">{formatFileSize(file?.meta?.size ?? file?.size ?? 0)}</div>
			</button>

			<!-- 删除按钮 -->
			<button
				type="button"
				class="p-1 rounded hover:bg-black/5 dark:hover:bg-white/10"
				on:click|stopPropagation={() => dispatch('delete', file.id)}
			>
				<GarbageBin className="size-3.5 text-gray-500 hover:text-red-500" />
			</button>
		</div>
	{/each}
</div>
