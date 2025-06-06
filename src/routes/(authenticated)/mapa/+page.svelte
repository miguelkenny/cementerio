<script lang="ts">
  import type { SupabaseClient } from "@supabase/supabase-js";
  import { onMount } from "svelte";
  import { writable, get } from "svelte/store";
  import NichoModal from "$lib/components/NichoModal.svelte";

  export let data: { supabase: SupabaseClient };
  const supabase = data.supabase;

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
    fecha_nacimiento: string | null;
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

  type Contrato = {
    fecha_inicio: string;
    duracion_anios: number;
    tipo_pago: string;
    monto_total: number;
  };

  let nichos = writable<Nicho[]>([]);
  let pasillos = writable<string[]>([]);
  let selectedNicho: Nicho | null = null;
  let fallecido: Fallecido;
  let responsable: Responsable;
  let contrato: Contrato = {
    fecha_inicio: new Date().toISOString().split("T")[0],
    duracion_anios: 1,
    tipo_pago: "total",
    monto_total: 0,
  };

  let showModal = false;
  let loading = false;

  onMount(async () => {
    const { data, error } = await supabase
      .from("nichos")
      .select("*")
      .order("pasillo")
      .order("fila")
      .order("columna");
    if (error) return console.error(error);
    nichos.set(data);
    pasillos.set([...new Set(data.map((n) => n.pasillo))]);
  });

  async function abrirModal(nicho: Nicho) {
    selectedNicho = nicho;
    fallecido = {
      nombre: "",
      fecha_nacimiento: null,
      fecha_fallecimiento: null,
      fecha_inhumacion: null,
      observaciones: null,
      responsable_id: "",
    };

    responsable = {
      nombre: "",
      dni: null,
      telefono: null,
      email: null,
    };

    if (nicho.estado === "ocupado") {
      const { data: fallecidoData, error: fErr } = await supabase
        .from("fallecidos")
        .select("*")
        .eq("nicho_id", nicho.id)
        .maybeSingle();
      if (fErr) return console.error(fErr);
      fallecido = fallecidoData;

      const { data: respData, error: rErr } = await supabase
        .from("responsables")
        .select("*")
        .eq("id", fallecido?.responsable_id)
        .maybeSingle();
      if (rErr) return console.error(rErr);
      responsable = respData;
    }

    showModal = true;
  }

  async function asignarNicho() {
    if (!selectedNicho || !fallecido || !responsable) return;
    loading = true;

    // Buscar o crear responsable
    let responsableId = null;
    if (responsable.dni) {
      const { data: existing, error: err1 } = await supabase
        .from("responsables")
        .select("id")
        .eq("dni", responsable.dni)
        .maybeSingle();

      if (err1) {
        loading = false;
        return console.error(err1);
      }

      if (existing) {
        responsableId = existing.id;
      } else {
        const { data: nuevoResp, error: err2 } = await supabase
          .from("responsables")
          .insert(responsable)
          .select()
          .single();
        if (err2) {
          loading = false;
          return console.error(err2);
        }
        responsableId = nuevoResp.id;
      }
    }

    // Insertar fallecido
    const fechaContratacion = contrato.fecha_inicio;
    const fechaVencimiento = new Date(
      new Date(fechaContratacion).setFullYear(
        new Date(fechaContratacion).getFullYear() + contrato.duracion_anios
      )
    )
      .toISOString()
      .split("T")[0];

    const montoPorCuota =
      contrato.tipo_pago === "cuotas"
        ? contrato.monto_total / (contrato.duracion_anios * 12)
        : contrato.monto_total;

    const { error: err3 } = await supabase.from("fallecidos").insert({
      ...fallecido,
      nicho_id: selectedNicho.id,
      responsable_id: responsableId,
      cremado: false,
      fecha_cremacion: null,
      fecha_contratacion: fechaContratacion,
      fecha_vencimiento: fechaVencimiento,
      tipo_contrato: contrato.tipo_pago,
      duracion_anios: contrato.duracion_anios,
      forma_pago: contrato.tipo_pago,
      monto_total: contrato.monto_total,
      monto_por_cuota: montoPorCuota,
    });

    if (err3) {
      loading = false;
      return console.error(err3);
    }

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

    if (err4) {
      loading = false;
      return console.error(err4);
    }

    // Marcar nicho como ocupado
    const { error: err5 } = await supabase
      .from("nichos")
      .update({ estado: "ocupado", ocupado: true })
      .eq("id", selectedNicho.id);

    if (err5) {
      loading = false;
      return console.error(err5);
    }

    // Refrescar nichos
    const { data: nuevos } = await supabase
      .from("nichos")
      .select("*")
      .order("pasillo")
      .order("fila")
      .order("columna");
    nichos.set(nuevos ?? []);
    pasillos.set([...new Set(nuevos?.map((n) => n.pasillo))]);

    showModal = false;
    loading = false;
  }

  function cerrarModal() {
    showModal = false;
  }
</script>

<h1 class="text-2xl font-bold mb-4">🗺️ Mapa de Nichos</h1>
{#each $pasillos as pasillo}
  <div class="mb-6">
    <h2 class="font-semibold">Pasillo {pasillo}</h2>
    <div class="grid grid-cols-10 gap-1">
      {#each $nichos.filter((n) => n.pasillo === pasillo) as nicho}
        <div
          on:click={() => abrirModal(nicho)}
          class={`p-2 text-xs text-center cursor-pointer rounded
            ${nicho.estado === "ocupado" ? "bg-red-400 text-white" : "bg-green-300 text-black"}
            hover:scale-105 transition`}
          title={`Fila: ${nicho.fila}, Columna: ${nicho.columna}`}
        >
          {nicho.fila}-{nicho.columna}
        </div>
      {/each}
    </div>
  </div>
{/each}

<NichoModal
  show={showModal}
  {selectedNicho}
  {fallecido}
  {responsable}
  {cerrarModal}
  {asignarNicho}
  {loading}
/>
