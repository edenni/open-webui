<script lang="ts">
	import GarbageBin from '$lib/components/icons/GarbageBin.svelte';
	import ConfirmDialog from '$lib/components/common/ConfirmDialog.svelte';
	import { formatFileSize } from '$lib/utils';
	import dayjs from 'dayjs';
	import { createEventDispatcher, getContext } from 'svelte';

	const i18n = getContext('i18n');

	const formatTimestamp = (timestamp: number) => {
		if (!timestamp) return '-';
		return dayjs.unix(timestamp).format('MMM DD, YYYY');
	};

	const formatTimeAgo = (timestamp: number) => {
		if (!timestamp) return '-';
		return dayjs.unix(timestamp).fromNow();
	};

	const getFileExtension = (filename: string) => {
		const ext = filename?.split('.').pop()?.toLowerCase();
		return ext || '';
	};

	const getFileIcon = (filename: string) => {
		const ext = getFileExtension(filename);
		const iconMap: { [key: string]: string } = {
			'pdf': '📄',
			'doc': '📝', 'docx': '📝',
			'xls': '📊', 'xlsx': '📊',
			'ppt': '📋', 'pptx': '📋',
			'txt': '📄', 'md': '📄',
			'jpg': '🖼️', 'jpeg': '🖼️', 'png': '🖼️', 'gif': '🖼️',
			'mp3': '🎵', 'wav': '🎵', 'mp4': '🎥',
			'zip': '📦', 'rar': '📦'
		};
		return iconMap[ext] || '📄';
	};

	const dispatch = createEventDispatcher();

	export let selectedFileId: string | null = null;
	export let files: any[] = [];
	export let small = false;

	let showDeleteConfirm = false;
	let pendingDeleteFileId: string | null = null;
	let pendingDeleteFilename: string = '';

	const handleDeleteClick = (file: any) => {
		pendingDeleteFileId = file.id;
		pendingDeleteFilename = file?.meta?.name ?? file?.name ?? 'Untitled';
		showDeleteConfirm = true;
	};

	const confirmDelete = () => {
		if (pendingDeleteFileId) {
			dispatch('delete', pendingDeleteFileId);
			resetDeleteState();
		}
	};

	const cancelDelete = () => {
		resetDeleteState();
	};

	const resetDeleteState = () => {
		showDeleteConfirm = false;
		pendingDeleteFileId = null;
		pendingDeleteFilename = '';
	};
</script>

<!-- Improved File List -->
<div class="w-full space-y-1">
	{#each files as file}
		{@const filename = file?.meta?.name ?? file?.name ?? 'Untitled'}
		{@const fileSize = formatFileSize(file?.meta?.size ?? file?.size ?? 0)}
		{@const isSelected = selectedFileId === file.id}
		
		<div
			class="group relative flex items-center p-3 rounded-lg transition-all duration-200 {isSelected 
				? 'bg-blue-50 dark:bg-blue-900/20 border border-blue-200 dark:border-blue-800' 
				: 'hover:bg-gray-50 dark:hover:bg-gray-800/50 border border-transparent hover:border-gray-200 dark:hover:border-gray-700'
			}"
		>
			<!-- File Icon -->
			<div class="flex-shrink-0 mr-3 text-lg">
				{getFileIcon(filename)}
			</div>
			
			<!-- File Info -->
			<div class="flex-1 min-w-0">
				<button
					type="button"
					class="w-full text-left focus:outline-none focus:ring-2 focus:ring-blue-500 rounded px-2 py-1 -mx-2 -my-1"
					on:click={() => dispatch('click', file.id)}
				>
					<!-- Filename -->
					<div class="font-medium text-sm text-gray-900 dark:text-gray-100 truncate">
						{filename}
					</div>
					
					<!-- Metadata -->
					<div class="flex items-center space-x-3 text-xs text-gray-500 dark:text-gray-400 mt-1">
						<span>{fileSize}</span>
						<span class="text-gray-300 dark:text-gray-600">•</span>
						<span title={formatTimestamp(file.created_at)}>
							{formatTimeAgo(file.created_at)}
						</span>
						{#if file?.meta?.chunks}
							<span class="text-gray-300 dark:text-gray-600">•</span>
							<span>{file.meta.chunks} chunks</span>
						{/if}
					</div>
				</button>
			</div>
			
			<!-- Delete Button -->
			<button
				type="button"
				class="opacity-0 group-hover:opacity-100 transition-opacity duration-200 p-1.5 rounded-md hover:bg-red-50 dark:hover:bg-red-900/20 focus:outline-none focus:ring-2 focus:ring-red-500"
				title="Delete file"
				on:click|stopPropagation={() => handleDeleteClick(file)}
			>
				<GarbageBin className="size-4 text-gray-400 hover:text-red-600 dark:hover:text-red-400" />
			</button>
		</div>
	{:else}
		<div class="text-center py-8 text-gray-500 dark:text-gray-400">
			<div class="text-4xl mb-2">📁</div>
			<div class="text-sm">No files in this knowledge base</div>
		</div>
	{/each}
</div>

<!-- Delete Confirmation Dialog -->
<ConfirmDialog
	bind:show={showDeleteConfirm}
	message={$i18n.t('Are you sure you want to delete "{{filename}}"? This action cannot be undone.', { filename: pendingDeleteFilename })}
	on:confirm={confirmDelete}
	on:cancel={cancelDelete}
/>