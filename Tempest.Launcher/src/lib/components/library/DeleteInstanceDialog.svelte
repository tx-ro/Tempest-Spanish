<script lang="ts">
	import { Trash2 } from "@lucide/svelte";
	import Modal from "$lib/components/ui/Modal.svelte";

	interface Props {
		open: boolean;
		instanceName: string;
		onconfirm: (deleteData: boolean) => Promise<void> | void;
	}

	let { open = $bindable(), instanceName, onconfirm }: Props = $props();

	let deleteData = $state(false);
	let isDeleting = $state(false);

	async function handleConfirm() {
		isDeleting = true;
		try {
			await onconfirm(deleteData);
		} finally {
			isDeleting = false;
			open = false;
		}
	}

	$effect(() => {
		if (!open) {
			deleteData = false;
			isDeleting = false;
		}
	});
</script>

<Modal bind:open title="Delete Instance">
	<div class="space-y-4">
		<p class="text-sm">
			¿Estás seguro que quieres eliminar <span class="font-semibold">"{instanceName}"</span>?
			Esta acción no puede ser revertida.
		</p>

		<div class="divider my-0"></div>

		<div class="form-control">
			<label class="label cursor-pointer justify-start gap-3">
				<input type="checkbox" class="checkbox checkbox-error" bind:checked={deleteData} />
				<div>
					<span class="label-text">Eliminar datos de la instancia del disco</span>
					<p class="text-xs opacity-60 mt-0.5">
						Elimina todos los archivos del juego de tu computadora
					</p>
				</div>
			</label>
		</div>
	</div>

	{#snippet actions()}
		<button class="btn btn-ghost" onclick={() => (open = false)} disabled={isDeleting}>
			Cancelar
		</button>
		<button class="btn btn-error" onclick={handleConfirm} disabled={isDeleting}>
			{#if isDeleting}
				<span class="loading loading-spinner loading-sm"></span>
				Borrando...
			{:else}
				<Trash2 size={16} />
				Borrar
			{/if}
		</button>
	{/snippet}
</Modal>
