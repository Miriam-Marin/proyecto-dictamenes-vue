<template>
  <!-- Barra de acciones -->
  <section class="modulo-acciones">
    <span class="modulo-acciones-titulo">Administrar Propietarios (MVZ)</span>

    <div class="modulo-acciones-botones">
      <button
        v-for="accion in acciones"
        :key="accion.id"
        type="button"
        class="sistpec-btn-accion"
        :class="{ active: selectedAction === accion.id }"
        @click="cambiarAccion(accion.id)"
      >
        {{ accion.label }}
      </button>
    </div>

    <div class="sistpec-info-box">
      <p class="sistpec-info-text">{{ descripcionAccionActual }}</p>
    </div>
  </section>

  <section class="modulo-contenido" ref="moduloContenidoRef">
    <!-- ALERTAS -->
    <div v-if="errores.length" class="modulo-alert modulo-alert--error">
      <ul>
        <li v-for="(e, i) in errores" :key="i">{{ e }}</li>
      </ul>
    </div>

    <div v-if="mensajeExito" class="modulo-alert modulo-alert--success">
      {{ mensajeExito }}
    </div>

    <!-- ====================== 1) REGISTRAR ====================== -->
    <div v-if="selectedAction === 'registrar'">
      <h3 class="subtitulo">Registrar propietario</h3>

      <div class="sistpec-info-box">
        <p class="sistpec-info-text">
          Capture los datos solicitados por el Comité. El RFC no es obligatorio.
          El estatus puede ser <strong>Activo</strong> o <strong>Finado</strong>.
        </p>
      </div>

      <form class="sistpec-form" @submit.prevent="guardarPropietario">
        <div class="sistpec-form-row">
          <div class="sistpec-form-group">
            <label>Apellido paterno</label>
            <input v-model="form.apellido_paterno" type="text" placeholder="Ej. Ruiz" />
          </div>

          <div class="sistpec-form-group">
            <label>Apellido materno</label>
            <input v-model="form.apellido_materno" type="text" placeholder="Ej. Mendoza" />
          </div>

          <div class="sistpec-form-group">
            <label>Nombre(s)</label>
            <input v-model="form.nombres" type="text" placeholder="Ej. Miguel Ángel" />
          </div>

          <div class="sistpec-form-group">
            <label>Estatus</label>
            <select v-model="form.estatus">
              <option value="Activo">Activo</option>
              <option value="Finado">Finado</option>
            </select>
          </div>
        </div>

        <div class="sistpec-form-row">
          <div class="sistpec-form-group">
            <label>CURP</label>
            <input v-model="form.curp" type="text" placeholder="Ej. RUMM690828HVZZNG03" />
          </div>

          <div class="sistpec-form-group">
            <label>Teléfono</label>
            <input v-model="form.telefono" type="text" placeholder="10 dígitos" />
          </div>

          <div class="sistpec-form-group" style="grid-column: span 2;">
            <label>Correo electrónico</label>
            <input v-model="form.correo" type="email" placeholder="Ej. correo@dominio.com" />
          </div>
        </div>

        <div class="sistpec-form-row">
          <div class="sistpec-form-group" style="grid-column: span 2;">
            <label>Domicilio</label>
            <input v-model="form.domicilio" type="text" placeholder="Calle, número, colonia..." />
          </div>

          <div class="sistpec-form-group">
            <label>Localidad</label>
            <input v-model="form.localidad" type="text" placeholder="Ej. Los Altos" />
          </div>

          <div class="sistpec-form-group">
            <label>Municipio</label>
            <input v-model="form.municipio" type="text" placeholder="Ej. Ayahualulco" />
          </div>
        </div>

        <div class="sistpec-form-row">
          <div class="sistpec-form-group">
            <label>Código Postal</label>
            <input v-model="form.codigo_postal" type="text" placeholder="Ej. 91260" />
          </div>

          <div class="sistpec-form-group">
            <label>Estado</label>
            <input v-model="form.estado" type="text" placeholder="Ej. Veracruz" />
          </div>

          <div class="sistpec-form-group" style="grid-column: span 2;">
            <label>Observaciones (opcional)</label>
            <input v-model="form.observaciones" type="text" placeholder="Notas" />
          </div>
        </div>

        <div class="sistpec-form-actions">
          <button type="submit" class="sistpec-btn-primary">GUARDAR</button>
          <button type="button" class="sistpec-btn-secondary" @click="limpiarForm">LIMPIAR</button>
        </div>
      </form>
    </div>

    <!-- ====================== 2) CONSULTAR ====================== -->
    <div v-else-if="selectedAction === 'consultar'">
      <h3 class="subtitulo">Consultar propietarios</h3>

      <div class="sistpec-search-bar">
        <div class="sistpec-form-group">
          <label>Nombre / Apellidos</label>
          <input v-model="filtros.nombre" type="text" placeholder="Ej. Ruiz Miguel" />
        </div>

        <div class="sistpec-form-group">
          <label>CURP</label>
          <input v-model="filtros.curp" type="text" placeholder="Ej. RUMM690828..." />
        </div>

        <div class="sistpec-form-group">
          <label>Municipio</label>
          <input v-model="filtros.municipio" type="text" placeholder="Ej. Ayahualulco" />
        </div>

        <div class="sistpec-form-group">
          <label>Estatus</label>
          <select v-model="filtros.estatus">
            <option value="">Todos</option>
            <option value="Activo">Activo</option>
            <option value="Finado">Finado</option>
          </select>
        </div>
      </div>

      <div class="sistpec-search-bar fechas-bar">
        <div class="sistpec-form-group">
          <label>Teléfono (opcional)</label>
          <input v-model="filtros.telefono" type="text" placeholder="Ej. 228..." />
        </div>

        <div class="sistpec-form-group sistpec-search-actions">
          <button type="button" class="sistpec-btn-primary" @click="buscar">BUSCAR</button>
          <button type="button" class="sistpec-btn-secondary" @click="limpiarBusqueda">LIMPIAR</button>
        </div>
      </div>

      <div v-if="buscado" class="sistpec-table-wrapper">
        <table class="sistpec-table">
          <thead>
            <tr>
              <th>Propietario</th>
              <th>CURP</th>
              <th>Teléfono</th>
              <th>Municipio</th>
              <th>Estatus</th>
              <th>Acciones</th>
            </tr>
          </thead>

          <tbody>
            <tr v-for="p in propietariosFiltrados" :key="p.id">
              <td>{{ p.apellido_paterno }} {{ p.apellido_materno }} {{ p.nombres }}</td>
              <td>{{ p.curp }}</td>
              <td>{{ p.telefono || '-' }}</td>
              <td>{{ p.municipio || '-' }}</td>
              <td>
                <span class="badge" :class="badgeEstatusClase(p.estatus)">
                  {{ p.estatus }}
                </span>
              </td>
              <td>
                <div class="acciones">
                  <button type="button" class="sistpec-btn-secondary sistpec-btn-sm" @click="verDetalle(p)">
                    VER
                  </button>

                  <button
                    type="button"
                    class="sistpec-btn-secondary sistpec-btn-sm"
                    :disabled="!puedeEditar(p)"
                    @click="abrirEdicion(p)"
                    title="Solo puede editar propietarios registrados por usted."
                  >
                    EDITAR
                  </button>

                  <button
                    type="button"
                    class="sistpec-btn-primary sistpec-btn-sm"
                    :disabled="p.estatus !== 'Finado'"
                    @click="abrirTraslado(p)"
                    title="Si está Finado, podrá trasladar sus UPP a otro propietario."
                  >
                    TRASLADO UPP
                  </button>
                </div>
              </td>
            </tr>

            <tr v-if="propietariosFiltrados.length === 0">
              <td colspan="6" class="sin-resultados">No se encontraron propietarios con esos criterios.</td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Panel detalle -->
      <div v-if="detalle" class="sistpec-edit-panel">
        <h4 class="subtitulo-secundario">Detalle: {{ nombreCompleto(detalle) }}</h4>

        <div class="detalle-grid">
          <div><span class="lbl">CURP:</span> {{ detalle.curp || '-' }}</div>
          <div><span class="lbl">Tel:</span> {{ detalle.telefono || '-' }}</div>
          <div style="grid-column: span 2;"><span class="lbl">Correo:</span> {{ detalle.correo || '-' }}</div>
          <div style="grid-column: span 2;"><span class="lbl">Domicilio:</span> {{ detalle.domicilio || '-' }}</div>
          <div><span class="lbl">Localidad:</span> {{ detalle.localidad || '-' }}</div>
          <div><span class="lbl">Municipio:</span> {{ detalle.municipio || '-' }}</div>
          <div><span class="lbl">CP:</span> {{ detalle.codigo_postal || '-' }}</div>
          <div><span class="lbl">Estado:</span> {{ detalle.estado || '-' }}</div>
          <div style="grid-column: span 2;"><span class="lbl">Obs:</span> {{ detalle.observaciones || '-' }}</div>
        </div>

        <div class="sistpec-form-actions">
          <button type="button" class="sistpec-btn-secondary" @click="detalle = null">CERRAR</button>
        </div>
      </div>
    </div>

    <!-- ====================== 3) EDITAR (PROPIOS) ====================== -->
    <div v-else-if="selectedAction === 'editar'">
      <h3 class="subtitulo">Editar propietario (solo propios)</h3>

      <div class="sistpec-info-box">
        <p class="sistpec-info-text">
          Solo puede editar propietarios que usted registró. Si cambia a <strong>Finado</strong>,
          el sistema permitirá trasladar sus UPP a otro propietario.
        </p>
      </div>

      <div v-if="!editando" class="modulo-alert modulo-alert--error">
        Seleccione un propietario desde <strong>CONSULTAR</strong> para editar.
      </div>

      <form v-else class="sistpec-form" @submit.prevent="guardarEdicion">
        <div class="sistpec-form-row">
          <div class="sistpec-form-group">
            <label>Apellido paterno</label>
            <input v-model="editando.apellido_paterno" type="text" />
          </div>
          <div class="sistpec-form-group">
            <label>Apellido materno</label>
            <input v-model="editando.apellido_materno" type="text" />
          </div>
          <div class="sistpec-form-group">
            <label>Nombre(s)</label>
            <input v-model="editando.nombres" type="text" />
          </div>
          <div class="sistpec-form-group">
            <label>Estatus</label>
            <select v-model="editando.estatus">
              <option value="Activo">Activo</option>
              <option value="Finado">Finado</option>
            </select>
          </div>
        </div>

        <div class="sistpec-form-row">
          <div class="sistpec-form-group">
            <label>CURP</label>
            <input v-model="editando.curp" type="text" />
          </div>
          <div class="sistpec-form-group">
            <label>Teléfono</label>
            <input v-model="editando.telefono" type="text" />
          </div>
          <div class="sistpec-form-group" style="grid-column: span 2;">
            <label>Correo electrónico</label>
            <input v-model="editando.correo" type="email" />
          </div>
        </div>

        <div class="sistpec-form-row">
          <div class="sistpec-form-group" style="grid-column: span 2;">
            <label>Domicilio</label>
            <input v-model="editando.domicilio" type="text" />
          </div>
          <div class="sistpec-form-group">
            <label>Localidad</label>
            <input v-model="editando.localidad" type="text" />
          </div>
          <div class="sistpec-form-group">
            <label>Municipio</label>
            <input v-model="editando.municipio" type="text" />
          </div>
        </div>

        <div class="sistpec-form-row">
          <div class="sistpec-form-group">
            <label>Código Postal</label>
            <input v-model="editando.codigo_postal" type="text" />
          </div>
          <div class="sistpec-form-group">
            <label>Estado</label>
            <input v-model="editando.estado" type="text" />
          </div>
          <div class="sistpec-form-group" style="grid-column: span 2;">
            <label>Observaciones</label>
            <input v-model="editando.observaciones" type="text" />
          </div>
        </div>

        <div class="sistpec-form-actions">
          <button type="submit" class="sistpec-btn-primary">GUARDAR CAMBIOS</button>
          <button type="button" class="sistpec-btn-secondary" @click="cancelarEdicion">CANCELAR</button>
        </div>
      </form>
    </div>

    <!-- ====================== 4) TRASLADAR UPP (DEMO) ====================== -->
    <div v-else-if="selectedAction === 'traslado'">
      <h3 class="subtitulo">Trasladar UPP de propietario finado</h3>

      <div class="sistpec-info-box">
        <p class="sistpec-info-text">
          Este flujo es para cuando un propietario está <strong>Finado</strong>. En producción, aquí se listarán
          sus UPP y se permitirá reasignarlas a otro propietario <strong>Activo</strong>.
        </p>
      </div>

      <div v-if="!propFinadoSeleccionado" class="modulo-alert modulo-alert--error">
        Seleccione un propietario (Finado) desde <strong>CONSULTAR</strong> y presione <strong>TRASLADO UPP</strong>.
      </div>

      <div v-else class="sistpec-edit-panel">
        <h4 class="subtitulo-secundario">Propietario finado: {{ nombreCompleto(propFinadoSeleccionado) }}</h4>

        <form class="sistpec-form" @submit.prevent="confirmarTrasladoDemo">
          <div class="sistpec-form-row">
            <div class="sistpec-form-group" style="grid-column: span 2;">
              <label>Nuevo propietario (Activo)</label>
              <select v-model="traslado.nuevo_propietario_id">
                <option value="" disabled>Seleccione</option>
                <option v-for="p in propietariosActivos" :key="p.id" :value="p.id">
                  {{ nombreCompleto(p) }}
                </option>
              </select>
            </div>

            <div class="sistpec-form-group" style="grid-column: span 2;">
              <label>Motivo / Observación (opcional)</label>
              <input v-model="traslado.observaciones" type="text" placeholder="Ej. Cambio de titular por fallecimiento" />
            </div>
          </div>

          <div class="sistpec-form-actions">
            <button type="submit" class="sistpec-btn-primary">CONFIRMAR (DEMO)</button>
            <button type="button" class="sistpec-btn-secondary" @click="cancelarTraslado">CANCELAR</button>
          </div>
        </form>

        <div class="sistpec-info-box" style="margin-top: 12px;">
          <p class="sistpec-info-text">
            Nota: aquí todavía no se hace el traslado real de UPP porque eso lo maneja el módulo de UPP (o backend).
            Este panel solo ilustra la regla de negocio.
          </p>
        </div>
      </div>
    </div>

    <!-- fallback -->
    <div v-else>
      <h3 class="subtitulo">Acción no disponible</h3>
      <p>El contenido para esta acción aún está en desarrollo.</p>
    </div>
  </section>
</template>

<script setup>
import { computed, nextTick, ref, watch } from 'vue';

defineProps({
  codigo: { type: String, required: false, default: '' },
  rol: { type: String, required: false, default: 'MVZ' }
});

/* ===================== Scroll ===================== */
const moduloContenidoRef = ref(null);
function scrollAlContenido() {
  nextTick(() => {
    if (!moduloContenidoRef.value) return;
    const rect = moduloContenidoRef.value.getBoundingClientRect();
    const offset = 90;
    const top = rect.top + window.scrollY - offset;
    window.scrollTo({ top, behavior: 'smooth' });
  });
}

/* ===================== Acciones ===================== */
const acciones = [
  { id: 'registrar', label: 'REGISTRAR' },
  { id: 'consultar', label: 'CONSULTAR' },
  { id: 'editar', label: 'EDITAR (PROPIOS)' },
  { id: 'traslado', label: 'TRASLADO UPP (FINADO)' }
];

const selectedAction = ref('registrar');
const errores = ref([]);
const mensajeExito = ref('');

function cambiarAccion(id) {
  selectedAction.value = id;
  scrollAlContenido();
}

const descripcionAccionActual = computed(() => {
  switch (selectedAction.value) {
    case 'registrar':
      return 'Registre propietarios con datos completos (CURP, contacto y domicilio).';
    case 'consultar':
      return 'Consulte propietarios registrados por usted, filtrando por CURP, municipio y estatus.';
    case 'editar':
      return 'Edite solo propietarios que usted registró.';
    case 'traslado':
      return 'Si un propietario está Finado, sus UPP pueden trasladarse a otro propietario Activo.';
    default:
      return '';
  }
});

watch(
  () => selectedAction.value,
  () => {
    errores.value = [];
    mensajeExito.value = '';
  }
);

/* ===================== DEMO: MVZ actual ===================== */
const mvzUserId = 10;

/* ===================== DEMO DATA ===================== */
const propietariosDemo = ref([
  {
    id: 1,
    mvz_user_id: 10,
    apellido_paterno: 'Ruiz',
    apellido_materno: 'Mendoza',
    nombres: 'Miguel Ángel',
    curp: 'RUMM690828HVZZNG03',
    telefono: '2282132079',
    correo: 'soto95.cavsegmail.com',
    domicilio: 'C. Niño Artillero S/N',
    municipio: 'Ayahualulco',
    localidad: 'Los Altos',
    codigo_postal: '91260',
    estado: 'Veracruz',
    estatus: 'Activo',
    observaciones: ''
  },
  {
    id: 2,
    mvz_user_id: 10,
    apellido_paterno: 'García',
    apellido_materno: 'López',
    nombres: 'Juan',
    curp: 'XXXX690828HVZZNG00',
    telefono: '',
    correo: '',
    domicilio: '',
    municipio: 'Xalapa',
    localidad: '',
    codigo_postal: '',
    estado: 'Veracruz',
    estatus: 'Finado',
    observaciones: 'Registro histórico'
  },
  {
    id: 3,
    mvz_user_id: 99,
    apellido_paterno: 'Otro',
    apellido_materno: 'MVZ',
    nombres: 'Ajeno',
    curp: 'YYYY690828HVZZNG00',
    telefono: '',
    correo: '',
    domicilio: '',
    municipio: 'Otro',
    localidad: '',
    codigo_postal: '',
    estado: 'Veracruz',
    estatus: 'Activo',
    observaciones: ''
  }
]);

/* ===================== Helpers ===================== */
function nombreCompleto(p) {
  return `${p.apellido_paterno || ''} ${p.apellido_materno || ''} ${p.nombres || ''}`.trim();
}

function badgeEstatusClase(estatus) {
  if (estatus === 'Activo') return 'badge--activo';
  if (estatus === 'Finado') return 'badge--inactivo';
  return 'badge--proceso';
}

/* ===================== 1) Registrar ===================== */
const form = ref({
  apellido_paterno: '',
  apellido_materno: '',
  nombres: '',
  curp: '',
  telefono: '',
  correo: '',
  domicilio: '',
  municipio: '',
  localidad: '',
  codigo_postal: '',
  estado: '',
  estatus: 'Activo',
  observaciones: ''
});

function limpiarForm() {
  form.value = {
    apellido_paterno: '',
    apellido_materno: '',
    nombres: '',
    curp: '',
    telefono: '',
    correo: '',
    domicilio: '',
    municipio: '',
    localidad: '',
    codigo_postal: '',
    estado: '',
    estatus: 'Activo',
    observaciones: ''
  };
}

function guardarPropietario() {
  errores.value = [];
  mensajeExito.value = '';

  const f = form.value;

  // Validaciones mínimas (ajustables)
  if (!String(f.apellido_paterno || '').trim()) errores.value.push('Capture el apellido paterno.');
  if (!String(f.apellido_materno || '').trim()) errores.value.push('Capture el apellido materno.');
  if (!String(f.nombres || '').trim()) errores.value.push('Capture el/los nombre(s).');
  if (!String(f.curp || '').trim()) errores.value.push('Capture la CURP.');
  if (!String(f.municipio || '').trim()) errores.value.push('Capture el municipio.');
  if (!String(f.estado || '').trim()) errores.value.push('Capture el estado.');
  if (errores.value.length) return;

  // CURP "única" por MVZ (misma CURP no debe duplicarse en registros del mismo MVZ)
  const curp = String(f.curp).trim().toUpperCase();
  const existe = propietariosDemo.value.some(p => p.mvz_user_id === mvzUserId && String(p.curp || '').toUpperCase() === curp);
  if (existe) return errores.value.push('Ya existe un propietario con esa CURP (registrado por usted).');

  propietariosDemo.value.push({
    id: Date.now(),
    mvz_user_id: mvzUserId,
    apellido_paterno: String(f.apellido_paterno || '').trim(),
    apellido_materno: String(f.apellido_materno || '').trim(),
    nombres: String(f.nombres || '').trim(),
    curp,
    telefono: String(f.telefono || '').trim(),
    correo: String(f.correo || '').trim(),
    domicilio: String(f.domicilio || '').trim(),
    municipio: String(f.municipio || '').trim(),
    localidad: String(f.localidad || '').trim(),
    codigo_postal: String(f.codigo_postal || '').trim(),
    estado: String(f.estado || '').trim(),
    estatus: f.estatus,
    observaciones: String(f.observaciones || '').trim()
  });

  mensajeExito.value = 'Propietario registrado (DEMO).';
  limpiarForm();
}

/* ===================== 2) Consultar ===================== */
const filtros = ref({ nombre: '', curp: '', municipio: '', estatus: '', telefono: '' });
const buscado = ref(false);
const detalle = ref(null);

function buscar() {
  buscado.value = true;
  errores.value = [];
  mensajeExito.value = '';
}

function limpiarBusqueda() {
  filtros.value = { nombre: '', curp: '', municipio: '', estatus: '', telefono: '' };
  buscado.value = false;
  detalle.value = null;
  errores.value = [];
  mensajeExito.value = '';
}

const propietariosFiltrados = computed(() => {
  if (!buscado.value) return [];
  const f = filtros.value;

  const n = f.nombre.trim().toLowerCase();
  const c = f.curp.trim().toUpperCase();
  const m = f.municipio.trim().toLowerCase();
  const e = f.estatus;
  const t = f.telefono.trim();

  const base = propietariosDemo.value.filter(p => p.mvz_user_id === mvzUserId);

  return base.filter(p => {
    const textoNombre = `${p.apellido_paterno} ${p.apellido_materno} ${p.nombres}`.toLowerCase();
    const okNombre = n ? textoNombre.includes(n) : true;
    const okCurp = c ? String(p.curp || '').toUpperCase().includes(c) : true;
    const okMun = m ? String(p.municipio || '').toLowerCase().includes(m) : true;
    const okEst = e ? p.estatus === e : true;
    const okTel = t ? String(p.telefono || '').includes(t) : true;
    return okNombre && okCurp && okMun && okEst && okTel;
  });
});

function verDetalle(p) {
  detalle.value = { ...p };
}

/* ===================== 3) Editar (propios) ===================== */
const editando = ref(null);

function puedeEditar(p) {
  return p?.mvz_user_id === mvzUserId;
}

function abrirEdicion(p) {
  errores.value = [];
  mensajeExito.value = '';
  if (!puedeEditar(p)) return errores.value.push('No tiene permisos para editar este propietario.');
  editando.value = { ...p };
  selectedAction.value = 'editar';
}

function cancelarEdicion() {
  editando.value = null;
  selectedAction.value = 'consultar';
}

function guardarEdicion() {
  errores.value = [];
  mensajeExito.value = '';

  if (!editando.value) return errores.value.push('No hay propietario seleccionado para editar.');
  if (!puedeEditar(editando.value)) return errores.value.push('No tiene permisos para editar este propietario.');

  const curp = String(editando.value.curp || '').trim().toUpperCase();
  if (!curp) errores.value.push('La CURP es obligatoria.');
  if (!String(editando.value.apellido_paterno || '').trim()) errores.value.push('Apellido paterno obligatorio.');
  if (!String(editando.value.apellido_materno || '').trim()) errores.value.push('Apellido materno obligatorio.');
  if (!String(editando.value.nombres || '').trim()) errores.value.push('Nombre(s) obligatorio.');
  if (!String(editando.value.municipio || '').trim()) errores.value.push('Municipio obligatorio.');
  if (!String(editando.value.estado || '').trim()) errores.value.push('Estado obligatorio.');
  if (errores.value.length) return;

  // evitar duplicar CURP con otro registro del mismo MVZ
  const existe = propietariosDemo.value.some(
    p => p.mvz_user_id === mvzUserId && p.id !== editando.value.id && String(p.curp || '').toUpperCase() === curp
  );
  if (existe) return errores.value.push('Ya existe otro propietario con esa CURP (registrado por usted).');

  const idx = propietariosDemo.value.findIndex(x => x.id === editando.value.id);
  if (idx === -1) return errores.value.push('No se encontró el propietario.');

  propietariosDemo.value[idx] = { ...propietariosDemo.value[idx], ...editando.value, curp };
  mensajeExito.value = 'Propietario actualizado (DEMO).';
  editando.value = null;
  selectedAction.value = 'consultar';
}

/* ===================== 4) Traslado UPP (DEMO) ===================== */
const propFinadoSeleccionado = ref(null);
const traslado = ref({ nuevo_propietario_id: '', observaciones: '' });

const propietariosActivos = computed(() =>
  propietariosDemo.value.filter(p => p.mvz_user_id === mvzUserId && p.estatus === 'Activo')
);

function abrirTraslado(p) {
  errores.value = [];
  mensajeExito.value = '';
  if (!puedeEditar(p)) return errores.value.push('No tiene permisos para este propietario.');
  if (p.estatus !== 'Finado') return errores.value.push('El propietario debe estar en estatus Finado para traslado.');
  propFinadoSeleccionado.value = { ...p };
  traslado.value = { nuevo_propietario_id: '', observaciones: '' };
  selectedAction.value = 'traslado';
}

function cancelarTraslado() {
  propFinadoSeleccionado.value = null;
  traslado.value = { nuevo_propietario_id: '', observaciones: '' };
  selectedAction.value = 'consultar';
}

function confirmarTrasladoDemo() {
  errores.value = [];
  mensajeExito.value = '';

  if (!propFinadoSeleccionado.value) return errores.value.push('No hay propietario finado seleccionado.');
  const nuevoId = Number(traslado.value.nuevo_propietario_id);
  if (!nuevoId) return errores.value.push('Seleccione el nuevo propietario.');

  const nuevo = propietariosDemo.value.find(p => p.id === nuevoId && p.mvz_user_id === mvzUserId && p.estatus === 'Activo');
  if (!nuevo) return errores.value.push('El nuevo propietario debe ser Activo y registrado por usted.');

  const ok = window.confirm(
    `CONFIRMAR (DEMO): trasladar UPP del propietario finado "${nombreCompleto(propFinadoSeleccionado.value)}" a "${nombreCompleto(nuevo)}"?`
  );
  if (!ok) return;

  mensajeExito.value = 'Traslado simulado (DEMO). El traslado real de UPP se realiza en el módulo UPP / backend.';
  cancelarTraslado();
}
</script>

<style scoped>
.modulo-acciones { margin-bottom: 20px; }
.modulo-acciones-titulo { display:block; font-size:14px; margin-bottom:8px; color:#333; font-weight:800; }
.modulo-acciones-botones { display:flex; flex-wrap:wrap; gap:6px; }

.sistpec-btn-accion{
  border:none; padding:8px 14px; font-size:12px; font-weight:800;
  text-transform:uppercase; border-radius:4px; cursor:pointer;
  background:#2f6b32; color:#fff; letter-spacing:0.4px;
}
.sistpec-btn-accion.active{ background:#244e26; }

.modulo-contenido { margin-top: 10px; }
.subtitulo { font-size:18px; margin:10px 0 15px; color:#333; }
.subtitulo-secundario { font-size:16px; margin:16px 0 10px; color:#333; }

.sistpec-info-box{
  margin-top:10px; padding:10px 14px; border-radius:4px;
  background:#e1f3e1; border:1px solid #c3e6c3;
}
.sistpec-info-text{ margin:0; font-size:13px; color:#225522; }

.modulo-alert{ margin-bottom:12px; padding:10px 14px; border-radius:4px; font-size:13px; }
.modulo-alert--error{ background:#fbeaea; border:1px solid #f5c2c2; color:#7a1f1f; }
.modulo-alert--success{ background:#e1f3e1; border:1px solid #c3e6c3; color:#225522; }

.sistpec-form{ display:flex; flex-direction:column; gap:16px; }
.sistpec-form-row{ display:grid; grid-template-columns: repeat(4, minmax(0, 1fr)); gap:12px; }

.sistpec-form-group{ display:flex; flex-direction:column; gap:4px; }
.sistpec-form-group label{ font-size:13px; font-weight:800; color:#444; }
.sistpec-form-group input,
.sistpec-form-group select{
  padding:8px 10px; border-radius:4px; border:1px solid #ccc;
  font-size:14px; outline:none;
}
.sistpec-form-group input:focus,
.sistpec-form-group select:focus{
  border-color:#2f6b32;
  box-shadow:0 0 0 1px rgba(47, 107, 50, 0.15);
}

.sistpec-form-actions{
  display:flex; justify-content:flex-end; gap:8px;
}

.sistpec-btn-primary{
  background:#2f6b32; color:#fff; border:none;
  padding:8px 18px; border-radius:4px;
  font-size:13px; font-weight:900; cursor:pointer;
}
.sistpec-btn-primary:hover{ background:#244e26; }
.sistpec-btn-primary:disabled{ opacity:.5; cursor:not-allowed; }

.sistpec-btn-secondary{
  background:#e0e0e0; color:#333; border:none;
  padding:8px 18px; border-radius:4px;
  font-size:13px; font-weight:900; cursor:pointer;
}
.sistpec-btn-secondary:hover{ background:#d0d0d0; }
.sistpec-btn-secondary:disabled{ opacity:.5; cursor:not-allowed; }

.sistpec-btn-sm{ padding:5px 10px; font-size:11px; }

.sistpec-search-bar{
  display:grid; grid-template-columns: repeat(4, minmax(0, 1fr));
  gap:12px; margin-bottom:16px;
}
.fechas-bar{ grid-template-columns: repeat(2, minmax(0, 1fr)); }
.sistpec-search-actions{
  display:flex; align-items:flex-end; gap:8px; justify-content:flex-end;
}

.sistpec-table-wrapper{ width:100%; overflow-x:auto; }
.sistpec-table{ width:100%; border-collapse:collapse; font-size:13px; }
.sistpec-table thead{ background:#7a061e; color:#fff; }
.sistpec-table th, .sistpec-table td{ padding:8px 10px; border:1px solid #ddd; text-align:left; }
.sistpec-table tbody tr:nth-child(even){ background:#fafafa; }
.sin-resultados{ text-align:center; color:#777; }

.sistpec-edit-panel{
  margin-top:20px; padding-top:10px; border-top:1px dashed #ccc;
}

.acciones{ display:flex; gap:6px; flex-wrap:wrap; }

.badge{ display:inline-block; padding:2px 8px; border-radius:10px; font-size:11px; font-weight:900; }
.badge--activo{ background:#e1f3e1; color:#225522; border:1px solid #c3e6c3; }
.badge--inactivo{ background:#fbeaea; color:#7a1f1f; border:1px solid #f5c2c2; }
.badge--proceso{ background:#fff4e5; color:#b26a00; border:1px solid #ffd7a3; }

.detalle-grid{
  display:grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap:8px 12px;
  font-size:13px;
}
.lbl{ font-weight:900; color:#444; }

@media (max-width: 768px) {
  .sistpec-search-bar { grid-template-columns: 1fr; }
  .fechas-bar { grid-template-columns: 1fr; }
  .sistpec-form-row { grid-template-columns: 1fr; }
  .detalle-grid { grid-template-columns: 1fr; }
}
</style>
