<script lang="ts">
  let { children } = $props();
</script>

<section class="min-h-screen bg-gray-50 p-6">
  <h1 class="text-xl font-bold mb-4 text-gray-700">
    Área protegida - Acceso autorizado
  </h1>
  <div class="bg-white rounded-xl shadow p-4">
    {@render children?.()}
  </div>
</section>
