<template>
  <HeaderNav admin-page />

  <div class="container-fluid">
    <div class="row min-vh-100">
      <aside class="col-12 col-md-3 col-lg-2 bg-light border-end p-3">
        <h2 class="h5 mb-3">Administración</h2>
        <div class="nav nav-pills flex-column gap-2">
          <button
            v-for="section in sections"
            :key="section"
            class="nav-link text-start"
            :class="{ active: currentSection === section }"
            type="button"
            @click="currentSection = section"
          >
            {{ section }}
          </button>
        </div>
      </aside>

      <main class="col p-4">
        <div v-if="errorMessage" class="alert alert-danger" role="alert">
          {{ errorMessage }}
        </div>
        <p v-if="loading" class="text-secondary">Cargando información...</p>

        <section v-if="currentSection === 'Servicios'">
          <div class="d-flex justify-content-between align-items-center mb-3">
            <h1 class="h2 mb-0">Servicios</h1>
            <button class="btn btn-primary" type="button" @click="newService">
              Agregar servicio
            </button>
          </div>

          <form
            v-if="showServiceForm"
            class="card card-body mb-3"
            @submit.prevent="saveService"
          >
            <div class="row g-3">
              <div class="col-md-4">
                <label class="form-label" for="service-name">Nombre</label>
                <input id="service-name" v-model="serviceForm.Nombre" class="form-control" required />
              </div>
              <div class="col-md-3">
                <label class="form-label" for="service-duration">Duración (minutos)</label>
                <input
                  id="service-duration"
                  v-model.number="serviceForm.Duracion"
                  class="form-control"
                  type="number"
                  min="1"
                  required
                />
              </div>
              <div class="col-md-3">
                <label class="form-label" for="service-price">Precio</label>
                <input
                  id="service-price"
                  v-model.number="serviceForm.Precio"
                  class="form-control"
                  type="number"
                  min="0"
                  required
                />
              </div>
              <div class="col-md-2 d-flex align-items-end gap-2">
                <button class="btn btn-success" type="submit" :disabled="saving">Guardar</button>
                <button class="btn btn-secondary" type="button" @click="showServiceForm = false">
                  Cancelar
                </button>
              </div>
            </div>
          </form>

          <div class="table-responsive">
            <table class="table table-striped align-middle">
              <thead>
                <tr>
                  <th>Nombre</th>
                  <th>Duración</th>
                  <th>Precio</th>
                  <th>Acciones</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="service in services" :key="service.idServicio">
                  <td>{{ service.Nombre }}</td>
                  <td>{{ service.Duracion }} min</td>
                  <td>{{ formatCurrency(service.Precio) }}</td>
                  <td>
                    <button
                      class="btn btn-sm btn-outline-primary me-2"
                      type="button"
                      @click="editService(service)"
                    >
                      Editar
                    </button>
                    <button
                      class="btn btn-sm btn-outline-danger"
                      type="button"
                      @click="deleteService(service)"
                    >
                      Eliminar
                    </button>
                  </td>
                </tr>
                <tr v-if="!loading && services.length === 0">
                  <td colspan="4" class="text-center">No hay servicios.</td>
                </tr>
              </tbody>
            </table>
          </div>
        </section>

        <section v-else-if="currentSection === 'Barberos'">
          <div class="d-flex justify-content-between align-items-center mb-3">
            <h1 class="h2 mb-0">Barberos</h1>
            <button class="btn btn-primary" type="button" @click="newBarber">
              Crear barbero
            </button>
          </div>

          <form
            v-if="showBarberForm"
            class="card card-body mb-3"
            @submit.prevent="saveBarber"
          >
            <div class="row g-3">
              <div class="col-md-3">
                <label class="form-label" for="barber-name">Nombre</label>
                <input id="barber-name" v-model="barberForm.Nombre" class="form-control" required />
              </div>
              <div class="col-md-3">
                <label class="form-label" for="barber-services">Especialidades</label>
                <select
                  id="barber-services"
                  v-model="barberForm.servicios"
                  class="form-select"
                  multiple
                >
                  <option
                    v-for="service in services"
                    :key="service.idServicio"
                    :value="service.idServicio"
                  >
                    {{ service.Nombre }}
                  </option>
                </select>
                <small class="form-text">Mantén Ctrl para elegir más de una.</small>
              </div>
              <div class="col-md-3">
                <label class="form-label" for="barber-phone">Teléfono</label>
                <input id="barber-phone" v-model="barberForm.telefono" class="form-control" required />
              </div>
              <div class="col-md-3">
                <label class="form-label" for="barber-email">Correo</label>
                <input
                  id="barber-email"
                  v-model="barberForm.correo"
                  class="form-control"
                  type="email"
                  required
                />
              </div>
              <div v-if="!barberForm.idUsuario" class="col-md-4">
                <label class="form-label" for="barber-password">Contraseña inicial</label>
                <input
                  id="barber-password"
                  v-model="barberForm.contrasenha"
                  class="form-control"
                  type="password"
                  minlength="6"
                  required
                />
              </div>
              <div class="col-12 d-flex gap-2">
                <button class="btn btn-success" type="submit" :disabled="saving">Guardar</button>
                <button class="btn btn-secondary" type="button" @click="showBarberForm = false">
                  Cancelar
                </button>
              </div>
            </div>
          </form>

          <div class="table-responsive">
            <table class="table table-striped align-middle">
              <thead>
                <tr>
                  <th>Nombre</th>
                  <th>Especialidad</th>
                  <th>Teléfono</th>
                  <th>Correo</th>
                  <th>Acciones</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="barber in barbers" :key="barber.idUsuario">
                  <td>{{ barber.Nombre }}</td>
                  <td>{{ serviceNames(barber.especialidad) }}</td>
                  <td>{{ barber.telefono }}</td>
                  <td>{{ barber.correo }}</td>
                  <td>
                    <button
                      class="btn btn-sm btn-outline-primary me-2"
                      type="button"
                      @click="editBarber(barber)"
                    >
                      Editar
                    </button>
                    <button
                      class="btn btn-sm btn-outline-danger"
                      type="button"
                      @click="deleteBarber(barber)"
                    >
                      Eliminar
                    </button>
                  </td>
                </tr>
                <tr v-if="!loading && barbers.length === 0">
                  <td colspan="5" class="text-center">No hay barberos registrados.</td>
                </tr>
              </tbody>
            </table>
          </div>
        </section>

        <section v-else-if="currentSection === 'Citas'">
          <h1 class="h2 mb-3">Citas</h1>
          <div class="row g-3 mb-4">
            <div class="col-sm-6 col-lg-3">
              <div class="card card-body">
                <span>Total</span><strong class="fs-3">{{ appointments.length }}</strong>
              </div>
            </div>
            <div class="col-sm-6 col-lg-3">
              <div class="card card-body">
                <span>Pendientes</span><strong class="fs-3">{{ pendingAppointments }}</strong>
              </div>
            </div>
            <div class="col-sm-6 col-lg-3">
              <div class="card card-body">
                <span>Realizadas</span><strong class="fs-3">{{ countAppointments('Finalizado') }}</strong>
              </div>
            </div>
            <div class="col-sm-6 col-lg-3">
              <div class="card card-body">
                <span>Canceladas</span><strong class="fs-3">{{ countAppointments('Cancelado') }}</strong>
              </div>
            </div>
          </div>

          <div class="table-responsive">
            <table class="table table-striped align-middle">
              <thead>
                <tr>
                  <th>Cliente</th>
                  <th>Servicio</th>
                  <th>Barbero</th>
                  <th>Fecha</th>
                  <th>Estado</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="appointment in appointments" :key="appointment.idCita">
                  <td>{{ appointment.cliente?.Nombre }}</td>
                  <td>{{ serviceNames(appointment.servicios) }}</td>
                  <td>{{ appointment.barbero?.Nombre }}</td>
                  <td>{{ appointment.Fecha_hora }}</td>
                  <td>
                    <select
                      class="form-select"
                      :value="appointment.estado"
                      aria-label="Estado de la cita"
                      @change="updateAppointmentStatus(appointment, $event.target.value)"
                    >
                      <option>Confirmado</option>
                      <option>Pendiente</option>
                      <option>Finalizado</option>
                      <option>Cancelado</option>
                    </select>
                  </td>
                </tr>
                <tr v-if="!loading && appointments.length === 0">
                  <td colspan="5" class="text-center">No hay citas.</td>
                </tr>
              </tbody>
            </table>
          </div>
        </section>

      </main>
    </div>
  </div>
</template>

<script setup>
import { computed, onMounted, ref } from 'vue'
import HeaderNav from '../components/HeaderNav.vue'

const sections = ['Servicios', 'Barberos', 'Citas']
const currentSection = ref('Servicios')
const loading = ref(false)
const saving = ref(false)
const errorMessage = ref('')

const services = ref([])
const barbers = ref([])
const appointments = ref([])

const serviceForm = ref({ idServicio: null, Nombre: '', Duracion: 30, Precio: 0 })
const showServiceForm = ref(false)
const barberForm = ref({
  idUsuario: null,
  Nombre: '',
  servicios: [],
  telefono: '',
  correo: '',
  contrasenha: '',
})
const showBarberForm = ref(false)

const pendingAppointments = computed(() =>
  appointments.value.filter((appointment) =>
    ['Pendiente', 'Confirmado'].includes(appointment.estado),
  ).length,
)

onMounted(loadData)

async function apiRequest(path, options = {}) {
  const token = localStorage.getItem('token')
  const response = await fetch(`${import.meta.env.VITE_API_URL}${path}`, {
    ...options,
    headers: {
      Accept: 'application/json',
      ...(options.body ? { 'Content-Type': 'application/json' } : {}),
      ...(token ? { Authorization: `Bearer ${token}` } : {}),
      ...options.headers,
    },
  })

  const result = await response.json()
  if (!response.ok) {
    throw new Error(result.message || `Error de API (${response.status})`)
  }
  return result
}

async function loadData() {
  loading.value = true
  errorMessage.value = ''

  try {
    const [servicesResult, usersResult, appointmentsResult] =
      await Promise.all([
        apiRequest('/servicios'),
        apiRequest('/usuarios'),
        apiRequest('/citas'),
      ])

    services.value = servicesResult.data
    barbers.value = usersResult.data.filter((user) => Number(user.Roles_IDRol) === 2)
    appointments.value = Array.isArray(appointmentsResult)
      ? appointmentsResult
      : appointmentsResult.data
  } catch (error) {
    errorMessage.value = error.message
  } finally {
    loading.value = false
  }
}

function formatCurrency(amount) {
  return new Intl.NumberFormat('es-MX', {
    style: 'currency',
    currency: 'MXN',
  }).format(Number(amount || 0))
}

function countAppointments(status) {
  return appointments.value.filter((appointment) => appointment.estado === status).length
}

function serviceNames(items) {
  return (items || []).map((service) => service.Nombre).join(', ')
}

function newService() {
  serviceForm.value = { idServicio: null, Nombre: '', Duracion: 30, Precio: 0 }
  showServiceForm.value = true
}

function editService(service) {
  serviceForm.value = { ...service }
  showServiceForm.value = true
}

async function saveService() {
  saving.value = true
  errorMessage.value = ''

  try {
    const editing = Boolean(serviceForm.value.idServicio)
    const path = editing ? `/servicios/${serviceForm.value.idServicio}` : '/servicios'
    await apiRequest(path, {
      method: editing ? 'PUT' : 'POST',
      body: JSON.stringify({
        Nombre: serviceForm.value.Nombre,
        Duracion: serviceForm.value.Duracion,
        Precio: serviceForm.value.Precio,
      }),
    })
    showServiceForm.value = false
    await loadData()
  } catch (error) {
    errorMessage.value = error.message
  } finally {
    saving.value = false
  }
}

async function deleteService(service) {
  if (!window.confirm(`¿Eliminar el servicio "${service.Nombre}"?`)) return
  errorMessage.value = ''

  try {
    await apiRequest(`/servicios/${service.idServicio}`, { method: 'DELETE' })
    await loadData()
  } catch (error) {
    errorMessage.value = error.message
  }
}

function newBarber() {
  barberForm.value = {
    idUsuario: null,
    Nombre: '',
    servicios: [],
    telefono: '',
    correo: '',
    contrasenha: '',
  }
  showBarberForm.value = true
}

function editBarber(barber) {
  barberForm.value = {
    idUsuario: barber.idUsuario,
    Nombre: barber.Nombre,
    servicios: (barber.especialidad || []).map((service) => service.idServicio),
    telefono: barber.telefono || '',
    correo: barber.correo || '',
    contrasenha: '',
  }
  showBarberForm.value = true
}

async function saveBarber() {
  saving.value = true
  errorMessage.value = ''
  const editing = Boolean(barberForm.value.idUsuario)
  const payload = {
    Nombre: barberForm.value.Nombre,
    correo: barberForm.value.correo,
    telefono: barberForm.value.telefono,
    Roles_IDRol: 2,
    servicios: barberForm.value.servicios,
    especialidad: services.value
      .filter((service) => barberForm.value.servicios.includes(service.idServicio))
      .map((service) => service.Nombre)
      .join(', '),
    terminos_aceptados: true,
  }
  if (!editing) payload.contrasenha = barberForm.value.contrasenha
  if (editing && barberForm.value.contrasenha) {
    payload.contrasenha = barberForm.value.contrasenha
  }

  try {
    await apiRequest(
      editing ? `/usuarios/${barberForm.value.idUsuario}` : '/usuarios',
      {
        method: editing ? 'PUT' : 'POST',
        body: JSON.stringify(payload),
      },
    )
    showBarberForm.value = false
    await loadData()
  } catch (error) {
    errorMessage.value = error.message
  } finally {
    saving.value = false
  }
}

async function deleteBarber(barber) {
  if (!window.confirm(`¿Eliminar el perfil de ${barber.Nombre}?`)) return
  errorMessage.value = ''

  try {
    await apiRequest(`/usuarios/${barber.idUsuario}`, { method: 'DELETE' })
    await loadData()
  } catch (error) {
    errorMessage.value = error.message
  }
}

async function updateAppointmentStatus(appointment, status) {
  errorMessage.value = ''
  try {
    await apiRequest(`/citas/${appointment.idCita}`, {
      method: 'PUT',
      body: JSON.stringify({ estado: status }),
    })
    appointment.estado = status
  } catch (error) {
    errorMessage.value = error.message
  }
}
</script>
