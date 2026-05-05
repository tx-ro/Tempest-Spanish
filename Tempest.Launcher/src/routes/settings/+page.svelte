<script lang="ts">
	import { Settings } from "@lucide/svelte";
	import { open } from "@tauri-apps/plugin-dialog";
	import { invalidateAll } from "$app/navigation";
	import Header from "$lib/components/ui/Header.svelte";
	import { createAboutInfoQuery } from "$lib/queries/about";
	import { defaultInstancePath, username } from "$lib/stores/settings";

	let activeTab = $state<"general" | "advanced">("general");

	let localUsername = $state($username);
	let localPath = $state($defaultInstancePath || "");

	let buildType = $state<string>(import.meta.env.DEV ? "Development" : "Production");

	let showSaveToast = $state(false);

	async function saveSettings() {
		username.set(localUsername);
		defaultInstancePath.set(localPath);
		await invalidateAll();

		showSaveToast = true;
		setTimeout(() => {
			showSaveToast = false;
		}, 1000);
	}

	async function browsePath() {
		const selected = await open({
			directory: true,
			multiple: false,
			title: "Seleccionar Directorio Por Defecto de Instancias",
		});

		if (selected) {
			localPath = selected;
		}
	}

	function resetAll() {
		localStorage.clear();
		location.href = "/";
	}

	const aboutQuery = createAboutInfoQuery();

	let appVersion = $derived(aboutQuery.data?.appVersion ?? "...");
	let osName = $derived(aboutQuery.data?.osName ?? "...");
	let architecture = $derived(aboutQuery.data?.architecture ?? "...");
	let buildDate = $derived(aboutQuery.data?.buildDate ?? "...");
</script>

<div class="flex flex-col h-full bg-base-100">
	<Header
		title="Configuración"
		tabs={[
			{ name: "General", value: "general" },
			{ name: "Advanced", value: "advanced" },
		]}
		{activeTab}
		onSelectTab={(tab) => (activeTab = tab)}
	>
		{#snippet icon()}
			<Settings size={32} class="opacity-60" />
		{/snippet}
		{#snippet subtitle()}
			<span>Configura tus preferencias de lanzamiento del launcher</span>
		{/snippet}
	</Header>

	<div class="flex-1 flex flex-col overflow-hidden bg-base-100">
		<div class="flex-1 overflow-y-auto">
			<div class="p-6">
				{#if activeTab === "general"}
					<div class="flex flex-col gap-4">
						<div class="form-control">
							<label for="username-input" class="label py-0.5">
								<span class="label-text text-sm">Nombre</span>
							</label>
							<input
								id="username-input"
								type="text"
								class="input input-bordered w-full"
								bind:value={localUsername}
								placeholder="Ingresa tu nombre de usuario"
							/>
						</div>

						<div class="form-control">
							<label for="path-input" class="label py-0.5">
								<span class="label-text text-sm">Directorio Por Defecto de Instancias</span>
							</label>
							<div class="join w-full">
								<input
									id="path-input"
									type="text"
									class="input input-bordered join-item flex-1 font-mono"
									bind:value={localPath}
									placeholder="Seleccionar directorio..."
								/>
								<button class="btn btn-accent join-item" onclick={browsePath}
									>Browse</button
								>
							</div>
						</div>

						<div class="flex justify-end gap-2 pt-2">
							<button class="btn btn-accent" onclick={saveSettings}
								>Guardar Cambios</button
							>
						</div>
					</div>
				{:else if activeTab === "advanced"}
					<div class="flex flex-col">
						<div class="flex flex-col gap-4">
							<h2 class="text-xl font-semibold text-error">Zona Peligrosa</h2>
							<p class="text-sm">
								Reestablecer va a borrar todos tus ajustes y devolverá todo a los
								valores por defecto. Esto no podrá ser revertido.
							</p>
							<div>
								<button class="btn btn-error" onclick={resetAll}
									>Reestablecer las configuraciones</button
								>
							</div>
						</div>

						<div class="divider"></div>

						<div class="flex flex-col gap-4">
							<h2 class="text-xl font-semibold">About</h2>
							<div class="flex flex-col gap-2 text-sm">
								<div class="flex justify-between">
									<span>Versión</span>
									<span class="font-mono">{appVersion}</span>
								</div>
								<div class="flex justify-between">
									<span>Entorno</span>
									<span class="font-mono">{buildType}</span>
								</div>
								<div class="flex justify-between">
									<span>Fecha de Construcción</span>
									<span class="font-mono">{buildDate}</span>
								</div>
								<div class="flex justify-between">
									<span>OS & Arch</span>
									<span class="font-mono">{architecture}-{osName}</span>
								</div>
							</div>
						</div>
					</div>
				{/if}
			</div>
		</div>
	</div>

	<!-- Save Toast Notification -->
	{#if showSaveToast}
		<div class="toast toast-end toast-top">
			<div class="alert alert-success">
				<span>Configuración guardada exitosamente</span>
			</div>
		</div>
	{/if}
</div>
