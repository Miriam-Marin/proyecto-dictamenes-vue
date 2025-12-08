<template>
  <div class="modulo-panel">
    <!-- Encabezado del módulo -->
    <header class="modulo-header">
      <h2 class="modulo-titulo">{{ titulo }}</h2>
      <p class="modulo-meta">
        Módulo: <strong>{{ codigo }}</strong>
        <span class="modulo-meta-sep">|</span>
        Rol: <strong>{{ rol }}</strong>
      </p>
    </header>

    <!-- Aquí se monta el componente específico según el módulo -->
    <component
      v-if="currentComponent"
      :is="currentComponent"
      :codigo="codigo"
      :rol="rol"
    />

    <!-- Fallback para módulos que aún no existen -->
    <section v-else class="modulo-contenido">
      <h3 class="subtitulo">En construcción</h3>
      <p>
        El contenido para el módulo
        <strong>{{ codigo }}</strong>
        aún está en desarrollo.
      </p>
    </section>
  </div>
</template>

<script setup>
import { computed } from 'vue';
import AdminUsuariosModulo from './AdminUsuariosModulo.vue';

const props = defineProps({
  titulo: { type: String, required: true },
  codigo: { type: String, required: true },
  rol:    { type: String, required: true }
});

// Aquí mapeamos códigos de módulo a componentes concretos
const componentMap = {
  adminAdministrarUsuarios: AdminUsuariosModulo
  // Más adelante:
  // adminAdministrarPropietarios: AdminPropietariosModulo,
  // adminAdministrarUPP: AdminUppModulo,
  // etc...
};

const currentComponent = computed(() => componentMap[props.codigo] || null);
</script>

<style scoped>
.modulo-panel {
  margin-top: 20px;
  padding: 30px;
  border-radius: 12px;
  background: #ffffff;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.08);
  text-align: left;
}

/* Encabezado */
.modulo-header {
  margin-bottom: 20px;
}

.modulo-titulo {
  margin: 0 0 5px 0;
  font-size: 22px;
  color: #7a061e;
}

.modulo-meta {
  margin: 0;
  font-size: 13px;
  color: #555;
}

.modulo-meta-sep {
  margin: 0 6px;
  color: #999;
}
</style>
