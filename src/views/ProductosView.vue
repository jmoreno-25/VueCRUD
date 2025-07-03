<script setup lang="ts"> 
import { ref, onMounted } from 'vue'

interface Cliente {
    CLI_CEDULA_RUC: string
    CLI_NOMBRE: string
    CLI_APELLIDO: string
    CLI_TELEFONO: string
    CLI_ESTADO: string
}

const productos = ref<Cliente[]>([])
const mostrarForm = ref(false)
const mensaje = ref('')
const esEdicion = ref(false)

const form = ref<Cliente>({
    CLI_CEDULA_RUC: '',
    CLI_NOMBRE: '',
    CLI_APELLIDO: '',
    CLI_TELEFONO: '',
    CLI_ESTADO: '',
})

const imagenesInput = ref('')

const fetchProductos = async () => {
    try {
        const res = await fetch('https://backendpeteats.runasp.net/api/clientes')
        productos.value = await res.json()
    } catch {
        alert('Error al cargar productos')
    }
}

const abrirCrear = () => {
    form.value = {
        CLI_CEDULA_RUC: '',
        CLI_NOMBRE: '',
        CLI_APELLIDO: '',
        CLI_TELEFONO: '',
        CLI_ESTADO: ''
    }
    esEdicion.value = false
    imagenesInput.value = ''
    mostrarForm.value = true
}

const editar = (p: Cliente) => {
    form.value = { ...p }
    esEdicion.value = true
    imagenesInput.value = ''
    mostrarForm.value = true
}

const eliminar = async (id: string) => {
    if (!confirm('Eliminar Cliente?')) return;
    await fetch(`https://backendpeteats.runasp.net/api/clientes/${id}`, { method: 'DELETE' });
    mensaje.value = 'Cliente eliminado exitosamente';
    setTimeout(() => {
        mensaje.value = '';
    }, 3000);
    fetchProductos();
}

const guardar = async () => {
    const url = esEdicion.value
        ? `https://backendpeteats.runasp.net/api/clientes/${form.value.CLI_CEDULA_RUC}`
        : `https://backendpeteats.runasp.net/api/clientes`;

    const method = esEdicion.value ? 'PUT' : 'POST';

    try {
        await fetch(url, {
            method,
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(form.value),
        });
        mensaje.value = esEdicion.value
            ? 'Cliente editado exitosamente'
            : 'Cliente creado exitosamente';
        setTimeout(() => {
            mensaje.value = '';
        }, 3000);
        fetchProductos();
        cerrarForm();
    } catch {
        mensaje.value = 'Error al guardar cliente';
    }
}

const cerrarForm = () => {
    mostrarForm.value = false
}
onMounted(fetchProductos)

</script>

<template>
    <div class="app">
        <h1 class="header">Gestion Clientes</h1>

        <p v-if="mensaje" class="mensaje-exito">{{ mensaje }}</p>

        <button @click="abrirCrear">Crear Cliente</button>

        <!--Tabla para mostrar productos-->
        <table>
            <thead>
                <tr>
                    <th>Cédula</th>
                    <th>Nombre</th>
                    <th>Apellido</th>
                    <th>Teléfono</th>
                    <th>Estado</th>
                    <th>Editar</th>
                    <th>Eliminar</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="p in productos" :key="p.CLI_CEDULA_RUC">
                    <td>{{ p.CLI_CEDULA_RUC }}</td>
                    <td>{{ p.CLI_NOMBRE }}</td>
                    <td>{{ p.CLI_APELLIDO }}</td>
                    <td>{{ p.CLI_TELEFONO }}</td>
                    <td>{{ p.CLI_ESTADO }}</td>
                    <td><button @click="editar(p)">Editar</button></td>
                    <td><button @click="eliminar(p.CLI_CEDULA_RUC)">Eliminar</button></td>
                </tr>
            </tbody>
        </table>

        <!--Formulario para crear o editar productos-->
        <!-- Modal centrado -->
        <div v-if="mostrarForm" class="modal-overlay">
            <div class="modal">
                <h2>{{ esEdicion ? 'Editar Cliente' : 'Crear Cliente' }}</h2>
                <input v-model="form.CLI_CEDULA_RUC" placeholder="Cédula/RUC" />
                <input v-model="form.CLI_NOMBRE" placeholder="Nombre" />
                <input v-model="form.CLI_APELLIDO" placeholder="Apellido" />
                <input type="text" v-model="form.CLI_TELEFONO" placeholder="Teléfono" />
                <input type="text" v-model="form.CLI_ESTADO" placeholder="Estado" />

                <button @click="guardar">Guardar</button>
                <button @click="cerrarForm">Cancelar</button>
                <p>{{ mensaje }}</p>
            </div>
        </div>
    </div>
</template>

<style scoped>
.app {
  max-width: 1100px;
  margin: 40px auto;
  padding: 24px;
  background: #f9fafb;
  border-radius: 18px;
  box-shadow: 0 4px 24px #0001;
}

.header {
  color: #2c3e50;
  text-align: center;
  margin-bottom: 24px;
  font-size: 2.2rem;
  letter-spacing: 1px;
}

button {
  background: #42b883;
  color: #fff;
  border: none;
  padding: 8px 18px;
  border-radius: 6px;
  margin: 4px;
  cursor: pointer;
  font-weight: 600;
  transition: background 0.2s;
}
button:hover {
  background: #2c8e6b;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin: 24px 0;
  background: #fff;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 2px 12px #0002;
}

th, td {
  padding: 14px 10px;
  text-align: center;
  color: #222;
  font-size: 1.08rem;
}

th {
  background: #e9f5f2;
  color: #2c3e50;
  font-size: 1.13rem;
  letter-spacing: 0.5px;
  border-bottom: 2px solid #42b88333;
  font-weight: 700;
}

td {
  font-weight: 500;
  background: #fff;
}

tbody tr {
  border-bottom: 1px solid #e0e0e0;
  transition: background 0.2s;
}
tbody tr:hover {
  background: #f1fdfb;
}

.modal-overlay {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(44, 62, 80, 0.35);
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal {
  background: #fff;
  border-radius: 14px;
  box-shadow: 0 8px 32px #0004;
  padding: 32px 24px;
  max-width: 400px;
  width: 90%;
  color: #222;
  z-index: 1001;
  position: relative;
  animation: modalIn 0.2s;
}

@keyframes modalIn {
  from { transform: scale(0.95); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}

.modal h2 {
  color: #42b883;
  margin-bottom: 18px;
}

.modal input,
.modal textarea {
  width: 100%;
  margin-bottom: 12px;
  padding: 8px;
  border-radius: 6px;
  border: 1px solid #bdbdbd;
  background: #f9fafb;
  color: #222;
  font-size: 1rem;
}

.mensaje-exito {
  background: #d4edda;
  color: #155724;
  border: 1px solid #c3e6cb;
  padding: 12px;
  margin-bottom: 20px;
  border-radius: 6px;
  font-weight: 600;
  text-align: center;
}

@media (max-width: 700px) {
  .app {
    padding: 8px;
  }
  table, th, td {
    font-size: 0.95rem;
    padding: 8px 4px;
  }
  .modal {
    padding: 16px 6px;
  }
}
</style>
