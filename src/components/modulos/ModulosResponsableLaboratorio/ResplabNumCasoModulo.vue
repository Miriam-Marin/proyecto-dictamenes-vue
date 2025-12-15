<template>
  <section class="modulo-acciones">
    <span class="modulo-acciones-titulo">Acciones disponibles</span>

    <div class="modulo-acciones-botones">
      <button
        v-for="a in acciones"
        :key="a.id"
        type="button"
        class="sistpec-btn-accion"
        :class="{ active: selectedAction === a.id }"
        @click="cambiarAccion(a.id)"
      >
        {{ a.label }}
      </button>
    </div>

    <div class="sistpec-info-box">
      <p class="sistpec-info-text">{{ descripcionAccionActual }}</p>
    </div>
  </section>

  <section class="modulo-contenido" ref="moduloContenidoRef">
    <div v-if="errores.length" class="modulo-alert modulo-alert--error">
      <ul><li v-for="(e,i) in errores" :key="i">{{ e }}</li></ul>
    </div>

    <div v-if="mensajeExito" class="modulo-alert modulo-alert--success">
      {{ mensajeExito }}
    </div>

    <!-- ASIGNAR -->
    <div v-if="selectedAction === 'asignar'">
      <h3 class="subtitulo">Asignar número de caso</h3>

      <form class="sistpec-form" @submit.prevent="asignarCaso">
        <div class="sistpec-form-row">
          <div class="sistpec-form-group">
            <label>Clave UPP</label>
            <input v-model="nuevoCaso.upp" type="text" required placeholder="Ej. VER-0001-2025" />
          </div>

          <div class="sistpec-form-group">
            <label>MVZ (quien captura)</label>
            <input v-model="nuevoCaso.mvz" type="text" required placeholder="Ej. MVZ Ana López" />
          </div>
        </div>

        <div class="sistpec-form-row">
          <div class="sistpec-form-group">
            <label>Propietario</label>
            <input v-model="nuevoCaso.propietario" type="text" required placeholder="Nombre completo" />
          </div>

          <div class="sistpec-form-group">
            <label>Fecha de apertura</label>
            <input v-model="nuevoCaso.fecha" type="date" required />
          </div>
        </div>

        <div class="sistpec-form-actions">
          <button class="sistpec-btn-primary" type="submit">ASIGNAR</button>
          <button class="sistpec-btn-secondary" type="button" @click="limpiarFormulario">LIMPIAR</button>
        </div>
      </form>
    </div>

    <!-- CONSULTAR -->
    <div v-else-if="selectedAction === 'consultar'">
      <header class="modulo-header">
        <h3 class="subtitulo">Consultar números de caso</h3>
        <p class="modulo-subtitle">Capture al menos un criterio de búsqueda.</p>
      </header>

      <div v-if="mostrarAlerta" class="modulo-alert modulo-alert--error">
        Debe capturar <strong>al menos un criterio de búsqueda</strong> para realizar la consulta.
      </div>

      <div class="sistpec-search-bar">
        <div class="sistpec-form-group">
          <label>Número de caso</label>
          <input v-model="filtros.caso" type="text" placeholder="Ej. BR25-001" />
        </div>
        <div class="sistpec-form-group">
          <label>UPP</label>
          <input v-model="filtros.upp" type="text" placeholder="Clave UPP" />
        </div>
        <div class="sistpec-form-group">
          <label>MVZ</label>
          <input v-model="filtros.mvz" type="text" placeholder="Nombre MVZ" />
        </div>
        <div class="sistpec-form-group">
          <label>Fecha</label>
          <input v-model="filtros.fecha" type="date" />
        </div>

        <div class="sistpec-form-group sistpec-search-actions">
          <button type="button" class="sistpec-btn-primary" @click="buscar">BUSCAR</button>
          <button type="button" class="sistpec-btn-secondary" @click="limpiarFiltros">LIMPIAR FILTROS</button>
        </div>
      </div>

      <div v-if="buscado" class="sistpec-table-wrapper">
        <table class="sistpec-table">
          <thead>
            <tr>
              <th>Número de caso</th>
              <th>Fecha</th>
              <th>UPP</th>
              <th>Propietario</th>
              <th>MVZ</th>
              <th>Estatus</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="c in casosFiltrados" :key="c.id">
              <td>{{ c.caso }}</td>
              <td>{{ c.fecha }}</td>
              <td>{{ c.upp }}</td>
              <td>{{ c.propietario }}</td>
              <td>{{ c.mvz }}</td>
              <td>
                <span class="badge" :class="c.estatus==='Pendiente' ? 'badge--proceso' : 'badge--activo'">
                  {{ c.estatus }}
                </span>
              </td>
            </tr>

            <tr v-if="casosFiltrados.length === 0">
              <td colspan="6" class="sin-resultados">No se encontraron coincidencias.</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <div v-else>
      <h3 class="subtitulo">Acción no disponible</h3>
    </div>
  </section>
</template>

<script setup>
import { computed, ref, nextTick } from 'vue';

defineProps({ codigo: String, rol: String });

const moduloContenidoRef = ref(null);
function scrollAlContenido() {
  nextTick(() => {
    if (!moduloContenidoRef.value) return;
    const offset = 90;
    const top = moduloContenidoRef.value.getBoundingClientRect().top + window.scrollY - offset;
    window.scrollTo({ top, behavior: 'smooth' });
  });
}

const acciones = [
  { id: 'asignar', label: 'ASIGNAR NÚMERO DE CASO' },
  { id: 'consultar', label: 'CONSULTAR NÚMEROS DE CASO' },
];

const selectedAction = ref('asignar');
const errores = ref([]);
const mensajeExito = ref('');
const buscado = ref(false);
const mostrarAlerta = ref(false);

function cambiarAccion(id) {
  selectedAction.value = id;
  errores.value = [];
  mensajeExito.value = '';
  buscado.value = false;
  mostrarAlerta.value = false;
  scrollAlContenido();
}

const demo = ref([
  { id: 1, caso: 'BR25-001', fecha: '2025-12-01', upp: 'VER-0001-2025', propietario: 'José López Ramírez', mvz: 'MVZ Ana López', estatus: 'Pendiente' },
  { id: 2, caso: 'BR25-002', fecha: '2025-12-03', upp: 'VER-0020-2025', propietario: 'María Hernández Torres', mvz: 'MVZ Juan Pérez', estatus: 'Concluido' },
]);

const nuevoCaso = ref({ upp: '', mvz: '', propietario: '', fecha: '' });

function asignarCaso() {
  errores.value = [];
  mensajeExito.value = '';

  const ok = window.confirm('¿Desea asignar el número de caso con los datos capturados?');
  if (!ok) return;

  const nextNum = String(demo.value.length + 1).padStart(3, '0');
  const caso = `BR25-${nextNum}`;

  demo.value.push({
    id: Date.now(),
    caso,
    fecha: nuevoCaso.value.fecha,
    upp: nuevoCaso.value.upp,
    propietario: nuevoCaso.value.propietario,
    mvz: nuevoCaso.value.mvz,
    estatus: 'Pendiente',
  });

  mensajeExito.value = `Número de caso asignado correctamente: ${caso}`;
  nuevoCaso.value = { upp: '', mvz: '', propietario: '', fecha: '' };
  scrollAlContenido();
}

function limpiarFormulario() {
  nuevoCaso.value = { upp: '', mvz: '', propietario: '', fecha: '' };
  errores.value = [];
  mensajeExito.value = '';
}

const filtros = ref({ caso: '', upp: '', mvz: '', fecha: '' });

function hayAlMenosUnFiltro() {
  const f = filtros.value;
  return !!(f.caso.trim() || f.upp.trim() || f.mvz.trim() || f.fecha);
}

function buscar() {
  mostrarAlerta.value = false;
  if (!hayAlMenosUnFiltro()) {
    buscado.value = false;
    mostrarAlerta.value = true;
    return;
  }
  buscado.value = true;
  scrollAlContenido();
}

function limpiarFiltros() {
  filtros.value = { caso: '', upp: '', mvz: '', fecha: '' };
  buscado.value = false;
  mostrarAlerta.value = false;
}

const casosFiltrados = computed(() => {
  if (!buscado.value) return [];

  const f = filtros.value;
  const caso = f.caso.trim().toLowerCase();
  const upp = f.upp.trim().toLowerCase();
  const mvz = f.mvz.trim().toLowerCase();
  const fecha = f.fecha;

  return demo.value.filter(c => {
    const okCaso = caso ? c.caso.toLowerCase().includes(caso) : true;
    const okUpp = upp ? c.upp.toLowerCase().includes(upp) : true;
    const okMvz = mvz ? c.mvz.toLowerCase().includes(mvz) : true;
    const okFec = fecha ? c.fecha === fecha : true;
    return okCaso && okUpp && okMvz && okFec;
  });
});
</script>

<style scoped>
/* Reutiliza el mismo estilo que ya tienes en otros módulos (igual al de AdminUsuarios). */
.modulo-acciones { 
    margin-bottom: 20px; 
}
.modulo-acciones-titulo { 
    display:block; 
    font-size:14px; 
    margin-bottom:8px; 
    color:#333; 
    font-weight:600; 
}
.modulo-acciones-botones { 
    display:flex; 
    flex-wrap:wrap; 
    gap:4px; 
}
.sistpec-btn-accion { 
    border:none; 
    padding:8px 16px; 
    font-size:12px; 
    font-weight:600; 
    text-transform:uppercase; 
    border-radius:3px; 
    cursor:pointer; 
    background:#2f6b32; 
    color:#fff; 
    letter-spacing:.5px; 
}
.sistpec-btn-accion.active { 
    background:#244e26; 
}

.sistpec-info-box { 
    margin-top:10px; 
    padding:10px 14px; 
    border-radius:4px; 
    background:#e1f3e1; 
    border:1px solid #c3e6c3; 
}
.sistpec-info-text { 
    margin:0; 
    font-size:13px; 
    color:#225522; 
}

.modulo-contenido { 
    margin-top:10px; 
}
.modulo-header { 
    margin-bottom:12px; 
    text-align:left; 
}
.modulo-subtitle { 
    margin:0; 
    font-size:13px; 
    color:#555; 
}
.subtitulo { 
    font-size:18px; 
    margin:10px 0 15px; 
    color:#333; 
}

.modulo-alert { 
    margin-bottom:12px; 
    padding:10px 14px; 
    border-radius:4px; 
    font-size:13px; 
}
.modulo-alert--error { 
    background:#fbeaea; 
    border:1px solid #f5c2c2; 
    color:#7a1f1f; 
}
.modulo-alert--success { 
    background:#e1f3e1; 
    border:1px solid #c3e6c3; 
    color:#225522; 
}

.sistpec-form { 
    display:flex; 
    flex-direction:column; 
    gap:16px; 
}
.sistpec-form-row { 
    display:grid; 
    grid-template-columns:repeat(2, minmax(0, 1fr)); 
    gap:16px; 
}
.sistpec-form-group { 
    display:flex; 
    flex-direction:column; 
    gap:4px; 
}
.sistpec-form-group label { 
    font-size:13px; 
    font-weight:600; 
    color:#444; 
}
.sistpec-form-group input { 
    padding:8px 10px; 
    border-radius:4px; 
    border:1px solid #ccc; 
    font-size:14px; 
    outline:none; 
}
.sistpec-form-actions { 
    display:flex; 
    justify-content:flex-end; 
    gap:8px; 
}

.sistpec-btn-primary { 
    background:#2f6b32; 
    color:#fff; 
    border:none; 
    padding:8px 18px; 
    border-radius:4px; 
    font-size:13px; 
    font-weight:600; 
    cursor:pointer; 
}
.sistpec-btn-primary:hover { 
    background:#244e26; 
}
.sistpec-btn-secondary { 
    background:#e0e0e0; 
    color:#333; 
    border:none; 
    padding:8px 18px; 
    border-radius:4px; 
    font-size:13px; 
    font-weight:600; 
    cursor:pointer; }
.sistpec-btn-secondary:hover { 
    background:#d0d0d0; 
}

.sistpec-search-bar { 
    display:grid; 
    grid-template-columns:repeat(4, minmax(0, 1fr)); 
    gap:12px; 
    margin-bottom:16px; 
}
.sistpec-search-actions { 
    grid-column: 1 / -1; 
    display:flex; justify-content:flex-end; 
    align-items:flex-end; 
    gap:8px; 
}

.sistpec-table-wrapper { 
    width:100%; 
    overflow-x:auto; 
}
.sistpec-table { 
    width:100%; 
    border-collapse:collapse; 
    font-size:13px; 
}
.sistpec-table thead { 
    background:#7a061e; 
    color:#fff; 
}
.sistpec-table th, 
.sistpec-table td { 
    padding:8px 10px; 
    border:1px solid #ddd; 
    text-align:left; 
}
.sistpec-table tbody tr:nth-child(even) { 
    background:#fafafa; 
}
.sin-resultados { 
    text-align:center; 
    color:#777; 
}

.badge { display:inline-block; 
    padding:2px 8px; 
    border-radius:10px; 
    font-size:11px; 
    font-weight:600; 
}
.badge--activo { 
    background:#e1f3e1; 
    color:#225522; 
}
.badge--proceso { 
    background:#fff4e5; 
    color:#b26a00; 
}

@media (max-width:768px){
  .sistpec-form-row { 
    grid-template-columns: 1fr;
 }
  .sistpec-search-bar { 
    grid-template-columns: 1fr;
 }
  .sistpec-search-actions { 
    grid-column:auto; 
    justify-content:stretch;
 }
}
</style>
