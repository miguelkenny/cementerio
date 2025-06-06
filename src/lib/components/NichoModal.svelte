<script lang="ts">
  export let show = false;
  export let selectedNicho;
  export let fallecido;
  export let responsable;
  export let cerrarModal: () => void;
  export let asignarNicho: () => void;
  export let loading = false;
</script>

{#if show}
  <div class="fixed inset-0 bg-black/50 flex items-center justify-center z-50">
    <div class="bg-white rounded-xl p-6 shadow-xl w-[90%] max-w-md relative">
      <button class="absolute top-2 right-2" on:click={cerrarModal}>✖</button>
      <h2 class="text-lg font-semibold mb-1">
        Nicho: {selectedNicho?.fila}-{selectedNicho?.columna}
      </h2>
      <hr class="mb-2" />
      <h2 class="text-md font-bold mb-1">Datos del Fallecido</h2>
      {#if selectedNicho?.estado === "ocupado"}
        <div class="text-sm space-y-2">
          <div>
            <strong>Fallecido:</strong>
            {fallecido?.nombre}<br />
            <strong>Falleció:</strong>
            {fallecido?.fecha_fallecimiento}<br />
            <strong>Inhumación:</strong>
            {fallecido?.fecha_inhumacion}<br />
            <strong>Observaciones:</strong>
            {fallecido?.observaciones ?? "—"}
          </div>
          <hr />
          <h2 class="text-md font-bold mb-2">Datos del Solicitante</h2>
          <div>
            <strong>Responsable:</strong>
            {responsable?.nombre}<br />
            <strong>DNI:</strong>
            {responsable?.dni}<br />
            <strong>Teléfono:</strong>
            {responsable?.telefono}<br />
            <strong>Email:</strong>
            {responsable?.email}
          </div>
        </div>
      {:else}
        <form class="space-y-2 text-sm">
          <input
            class="w-full border rounded p-2"
            bind:value={fallecido.nombre}
            placeholder="Nombre fallecido"
          />
          <input
            class="w-full border rounded p-2"
            type="date"
            bind:value={fallecido.fecha_fallecimiento}
            placeholder="Fecha fallecimiento"
          />
          <input
            class="w-full border rounded p-2"
            type="date"
            bind:value={fallecido.fecha_inhumacion}
            placeholder="Fecha inhumación"
          />
          <textarea
            class="w-full border rounded p-2"
            bind:value={fallecido.observaciones}
            placeholder="Observaciones"
          ></textarea>
          <hr />
          <h2 class="text-md font-bold mb-2">Datos del Solicitante</h2>
          <input
            class="w-full border rounded p-2"
            bind:value={responsable.nombre}
            placeholder="Nombre responsable"
          />
          <input
            class="w-full border rounded p-2"
            bind:value={responsable.dni}
            placeholder="DNI"
          />
          <input
            class="w-full border rounded p-2"
            bind:value={responsable.telefono}
            placeholder="Teléfono"
          />
          <input
            class="w-full border rounded p-2"
            bind:value={responsable.email}
            placeholder="Email"
          />
          <button
            type="button"
            class="bg-blue-600 text-white py-2 px-4 rounded w-full mt-2"
            on:click={asignarNicho}
            disabled={loading}
          >
            {loading ? "Guardando..." : "Asignar nicho"}
          </button>
        </form>
      {/if}
    </div>
  </div>
{/if}
