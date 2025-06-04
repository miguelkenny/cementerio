<script lang="ts">
    import { onMount } from "svelte";
    import {
        PUBLIC_SUPABASE_URL,
        PUBLIC_SUPABASE_ANON_KEY,
    } from "$env/static/public";
    import { createClient } from "@supabase/supabase-js";

    const supabase = createClient(
        PUBLIC_SUPABASE_URL,
        PUBLIC_SUPABASE_ANON_KEY,
    );
    type Contrato = {
        fecha_inicio: string;
        duracion_anios: number;
        tipo_pago: string;
        monto_total: number;
    };

    let contrato: Contrato = {
        fecha_inicio: new Date().toISOString().split("T")[0],
        duracion_anios: 1,
        tipo_pago: "total",
        monto_total: 0,
    };

    type Nicho = {
        id: string;
        pasillo: string;
        fila: number;
        columna: number;
        estado: string;
        ocupado: boolean;
    };

    type Fallecido = {
        nombre: string;
        fecha_fallecimiento: string | null;
        fecha_inhumacion: string | null;
        observaciones: string | null;
        responsable_id: string;
    };

    type Responsable = {
        nombre: string;
        dni: string | null;
        telefono: string | null;
        email: string | null;
    };

    let nichos: Nicho[] = [];
    let pasillos: string[] = [];
    let selectedNicho: Nicho | null = null;
    let fallecido: Fallecido;
    let responsable: Responsable;
    let showModal = false;

    onMount(async () => {
        const { data, error } = await supabase.from("nichos").select("*");
        if (error) return console.error(error);
        nichos = data;
        pasillos = [...new Set(nichos.map((n) => n.pasillo))].sort();
    });

    async function asignarNicho() {
        if (!selectedNicho || !fallecido || !responsable) return;

        // Buscar responsable por DNI (si existe)
        let responsableId: string | null = null;
        if (responsable.dni) {
            const { data: existingResp, error: err } = await supabase
                .from("responsables")
                .select("id")
                .eq("dni", responsable.dni)
                .maybeSingle();

            if (err) return console.error("Error buscando responsable", err);

            if (existingResp) {
                responsableId = existingResp.id;
            } else {
                // Crear nuevo responsable
                const { data: newResp, error: err2 } = await supabase
                    .from("responsables")
                    .insert(responsable)
                    .select()
                    .single();

                if (err2)
                    return console.error("Error creando responsable", err2);
                responsableId = newResp.id;
            }
        }

        // Crear fallecido
        const { data: fallecidoCreated, error: err3 } = await supabase
            .from("fallecidos")
            .insert({
                ...fallecido,
                nicho_id: selectedNicho.id,
                responsable_id: responsableId,
            })
            .select()
            .single();

        if (err3) return console.error("Error creando fallecido", err3);

        // Crear contrato
        const { error: err4 } = await supabase.from("contratos").insert({
            responsable_id: responsableId,
            nicho_id: selectedNicho.id,
            fecha_inicio: contrato.fecha_inicio,
            duracion_anios: contrato.duracion_anios,
            tipo_pago: contrato.tipo_pago,
            monto_total: contrato.monto_total,
            estado: "activo",
        });

        if (err4) return console.error("Error creando contrato", err4);

        // Actualizar estado del nicho
        const { error: err5 } = await supabase
            .from("nichos")
            .update({ estado: "ocupado" })
            .eq("id", selectedNicho.id);

        if (err5) return console.error("Error actualizando nicho", err5);

        // Refrescar mapa y cerrar
        const { data: actualizados } = await supabase
            .from("nichos")
            .select("*");
        nichos = actualizados ?? [];
        pasillos = [...new Set(nichos.map((n) => n.pasillo))].sort();

        showModal = false;
    }

    async function abrirModal(nicho: Nicho) {
        selectedNicho = nicho;
        fallecido = {};
        responsable = {};
        showModal = true;

        if (nicho.estado === "ocupado") {
            const { data: fallecidos, error: err1 } = await supabase
                .from("fallecidos")
                .select("*")
                .eq("nicho_id", nicho.id)
                .maybeSingle();

            if (err1) return console.error(err1);
            fallecido = fallecidos;

            if (fallecido?.responsable_id) {
                const { data: resp, error: err2 } = await supabase
                    .from("responsables")
                    .select("*")
                    .eq("id", fallecido.responsable_id)
                    .maybeSingle();

                if (err2) return console.error(err2);
                responsable = resp;
            }
        }
    }

    function cerrarModal() {
        showModal = false;
    }
</script>

<h1 class="text-2xl font-bold mb-4">🗺️ Mapa de Nichos</h1>

{#each pasillos as pasillo}
    <div class="mb-8">
        <h2 class="text-lg font-semibold mb-2">Pasillo {pasillo}</h2>
        <div
            class="grid gap-1"
            style="grid-template-columns: repeat(10, minmax(0, 1fr));"
        >
            {#each nichos.filter((n) => n.pasillo === pasillo) as nicho}
                <div
                    on:click={() => abrirModal(nicho)}
                    title={`Fila: ${nicho.fila}, Col: ${nicho.columna}, Estado: ${nicho.estado}`}
                    class={`h-10 rounded flex items-center justify-center text-xs font-semibold cursor-pointer 
            ${nicho.estado === "ocupado" ? "bg-red-400 text-white" : "bg-green-300 text-gray-800"} 
            hover:scale-105 transition-transform`}
                >
                    {nicho.fila}-{nicho.columna}
                </div>
            {/each}
        </div>
    </div>
{/each}

{#if showModal}
    <div
        class="fixed inset-0 bg-black/50 flex items-center justify-center z-50"
    >
        <div
            class="bg-white rounded-xl shadow-lg p-6 w-[90%] max-w-md relative"
        >
            <button
                class="absolute top-2 right-2 text-xl"
                on:click={cerrarModal}>✖️</button
            >
            <h3 class="text-lg font-semibold mb-2">
                Nicho: {selectedNicho?.fila}-{selectedNicho?.columna}
            </h3>
            {#if selectedNicho?.estado === "ocupado"}
                <div class="space-y-2 text-sm">
                    {#if fallecido}
                        <div>
                            <strong>Fallecido:</strong>
                            {fallecido.nombre}<br />
                            <strong>Falleció:</strong>
                            {fallecido.fecha_fallecimiento}<br />
                            <strong>Inhumación:</strong>
                            {fallecido.fecha_inhumacion}<br />
                            <strong>Obs:</strong>
                            {fallecido.observaciones ?? "—"}
                        </div>
                    {/if}
                    {#if responsable}
                        <div>
                            <strong>Responsable:</strong>
                            {responsable.nombre}<br />
                            <strong>DNI:</strong>
                            {responsable.dni}<br />
                            <strong>Tel:</strong>
                            {responsable.telefono}<br />
                            <strong>Email:</strong>
                            {responsable.email}
                        </div>
                    {/if}
                </div>
            {:else}
                <form
                    on:submit|preventDefault={asignarNicho}
                    class="grid md:grid-cols-2 gap-4 text-sm"
                >
                    <div>
                        <h4 class="font-semibold mb-1">Fallecido</h4>
                        <input
                            class="input w-full mb-1"
                            placeholder="Nombre"
                            type="text"
                            bind:value={fallecido.nombre}
                        />
                        <label>Fecha Fallecido</label>
                        <input
                            class="input w-full mb-1"
                            type="date"
                            bind:value={fallecido.fecha_fallecimiento}
                        />
                        <label>Fecha Inhumación</label>
                        <input
                            class="input w-full"
                            type="date"
                            bind:value={fallecido.fecha_inhumacion}
                            placeholder="Fecha inhumación"
                        />
                    </div>
                    <div>
                        <h4 class="font-semibold mb-1">Responsable</h4>
                        <input
                            class="input w-full mb-1"
                            placeholder="Nombre"
                            bind:value={responsable.nombre}
                        />
                        <input
                            class="input w-full mb-1"
                            placeholder="DNI"
                            bind:value={responsable.dni}
                        />
                        <input
                            class="input w-full mb-1"
                            placeholder="Teléfono"
                            bind:value={responsable.telefono}
                        />
                        <input
                            class="input w-full"
                            placeholder="Email"
                            bind:value={responsable.email}
                        />
                    </div>
                    <div class="md:col-span-2">
                        <h4 class="font-semibold mb-1">Contrato</h4>
                        <label>Fecha Inicio</label>
                        <input
                            class="input w-full mb-1"
                            type="date"
                            bind:value={contrato.fecha_inicio}
                        />
                        <select
                            class="input w-full mb-1"
                            bind:value={contrato.tipo_pago}
                        >
                            <option value="mensual">Mensual</option>
                            <option value="anual">Anual</option>
                            <option value="total">Total</option>
                        </select>
                        <label>Duracion Años</label>
                        <input
                            class="input w-full mb-1"
                            type="number"
                            min="1"
                            max="5"
                            placeholder="Duración en años"
                            bind:value={contrato.duracion_anios}
                        />
                        <label>Monto</label>
                        <input
                            class="input w-full"
                            type="number"
                            placeholder="Monto total"
                            bind:value={contrato.monto_total}
                        />
                    </div>
                    <div class="md:col-span-2 flex justify-end">
                        <button
                            type="submit"
                            class="bg-blue-600 text-white px-4 py-2 rounded"
                            >Confirmar asignación</button
                        >
                    </div>
                </form>
            {/if}
        </div>
    </div>
{/if}

<style>
  .input {
    @apply border border-gray-300 rounded px-2 py-1 text-sm;
  }
</style>