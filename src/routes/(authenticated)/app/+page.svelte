<script lang="ts">
  import type { SupabaseClient } from "@supabase/supabase-js";
  import { onMount } from "svelte";
  import { writable } from "svelte/store";

  export let data: { supabase: SupabaseClient };

  const supabase: SupabaseClient = data.supabase;
  let totalNichos = 101;

  // Datos de ejemplo (mock)
  let resumen = writable({
    ocupados: 0,
    libres: 0,
    enMora: 56,
    totalMes: "$150.000",
  });

  const pagosRecientes = [
    {
      fecha: "2025-06-01",
      nicho: "A-12",
      fallecido: "María Rodríguez",
      monto: "$5.000",
    },
    {
      fecha: "2025-05-29",
      nicho: "C-3",
      fallecido: "Carlos Mendoza",
      monto: "$4.000",
    },
    {
      fecha: "2025-05-27",
      nicho: "B-8",
      fallecido: "Lucía Gómez",
      monto: "$6.000",
    },
    {
      fecha: "2025-05-25",
      nicho: "D-2",
      fallecido: "Miguel Pérez",
      monto: "$4.500",
    },
    {
      fecha: "2025-05-23",
      nicho: "A-4",
      fallecido: "Ana Torres",
      monto: "$5.000",
    },
  ];
  async function nichosOcupados() {
    const { data: nichos, error } = await supabase
      .from("nichos")
      .select("*")
      .eq("ocupado", true);

    if (error) {
      console.error("Error al obtener Nichos:", error.message);
      return;
    }
    let ocupados = nichos.length;
    let libres = totalNichos - ocupados;

    resumen.set({
      ocupados,
      libres,
      enMora: 56,
      totalMes: "$150.000",
    });
  }
  onMount(() => {
    nichosOcupados();
  });
</script>

<div class="p-6 space-y-6">
  <h1 class="text-2xl font-bold">
    Dashboard - Cementerio Parque Cerro Del Oeste S.R.L.
  </h1>
  <!-- Atajos rápidos -->
  <div class="grid grid-cols-2 sm:grid-cols-4 gap-4">
    <a
      href="/mapa"
      class="bg-green-100 text-green-700 font-semibold py-3 px-4 rounded-xl shadow hover:bg-green-200 text-center"
    >
      🗺️ Ver mapa de nichos
    </a>
    <a
      href="/fallecidos/nuevo"
      class="bg-blue-100 text-blue-700 font-semibold py-3 px-4 rounded-xl shadow hover:bg-blue-200 text-center"
    >
      🗺️ Ver mapa de parcelas
    </a>
    <a
      href="/pagos"
      class="bg-yellow-100 text-yellow-700 font-semibold py-3 px-4 rounded-xl shadow hover:bg-yellow-200 text-center"
    >
      📄 Ver lista de pagos
    </a>
    <a
      href="/reportes"
      class="bg-purple-100 text-purple-700 font-semibold py-3 px-4 rounded-xl shadow hover:bg-purple-200 text-center"
    >
      📊 Reportes mensuales
    </a>
  </div>
  <!-- Cards resumen -->
  <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
    <div class="bg-white rounded-2xl shadow p-4">
      <p class="text-gray-500">Nichos ocupados</p>
      <p class="text-2xl font-semibold">{$resumen.ocupados}</p>
    </div>
    <div class="bg-white rounded-2xl shadow p-4">
      <p class="text-gray-500">Nichos disponibles</p>
      <p class="text-2xl font-semibold">{$resumen.libres}</p>
    </div>
    <div class="bg-white rounded-2xl shadow p-4">
      <p class="text-gray-500">En mora</p>
      <p class="text-2xl font-semibold text-red-500">{$resumen.enMora}</p>
    </div>
    <div class="bg-white rounded-2xl shadow p-4">
      <p class="text-gray-500">Recaudación mensual</p>
      <p class="text-2xl font-semibold text-green-600">
        {$resumen.totalMes}
      </p>
    </div>
  </div>

  <!-- Últimos pagos -->
  <div class="bg-white rounded-2xl shadow p-4">
    <h2 class="text-lg font-bold mb-4">Últimos pagos</h2>
    <table class="w-full text-sm">
      <thead>
        <tr class="text-left border-b">
          <th class="pb-2">Fecha</th>
          <th class="pb-2">Nicho</th>
          <th class="pb-2">Fallecido</th>
          <th class="pb-2">Monto</th>
        </tr>
      </thead>
      <tbody>
        {#each pagosRecientes as pago}
          <tr class="border-b hover:bg-gray-50">
            <td class="py-2">{pago.fecha}</td>
            <td>{pago.nicho}</td>
            <td>{pago.fallecido}</td>
            <td>{pago.monto}</td>
          </tr>
        {/each}
      </tbody>
    </table>
  </div>
</div>

<style>
  :global(body) {
    @apply bg-gray-100;
  }
</style>
