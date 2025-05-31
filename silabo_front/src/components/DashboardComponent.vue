<template>
  <div class="dashboard-container">
    <!-- Menú lateral -->
    <aside class="sidebar" :class="{ 'sidebar-collapsed': sidebarCollapsed }">
      <div class="sidebar-header">
        <h3 v-if="!sidebarCollapsed">Mi App</h3>
        <button @click="toggleSidebar" class="toggle-btn">
          {{ sidebarCollapsed ? '→' : '←' }}
        </button>
      </div>
      
      <nav class="sidebar-nav">
        <ul>
          <li>
            <a href="#" @click="setActiveSection('dashboard')" 
               :class="{ active: activeSection === 'dashboard' }">
              <span class="icon">🏠</span>
              <span v-if="!sidebarCollapsed" class="text">Dashboard</span>
            </a>
          </li>
          <li>
            <a href="#" @click="setActiveSection('estudiantes')" 
               :class="{ active: activeSection === 'estudiantes' }">
              <span class="icon">👨‍🎓</span>
              <span v-if="!sidebarCollapsed" class="text">Estudiantes</span>
            </a>
          </li>
          <li>
            <a href="#" @click="setActiveSection('actividad')" 
               :class="{ active: activeSection === 'actividad' }">
              <span class="icon">📋</span>
              <span v-if="!sidebarCollapsed" class="text">Actividad</span>
            </a>
          </li>
          <li>
            <a href="#" @click="setActiveSection('universidad')" 
               :class="{ active: activeSection === 'universidad' }">
              <span class="icon">🏛️</span>
              <span v-if="!sidebarCollapsed" class="text">Universidad</span>
            </a>
          </li>
        </ul>
      </nav>

      <div class="sidebar-footer">
        <button @click="handleLogout" class="logout-btn">
          <span class="icon">🚪</span>
          <span v-if="!sidebarCollapsed" class="text">Cerrar Sesión</span>
        </button>
      </div>
    </aside>

    <!-- Contenido principal -->
    <main class="main-content">
      <header class="main-header">
        <h1>{{ getSectionTitle() }}</h1>
        <div class="user-info">
          <span>Bienvenido, Usuario</span>
        </div>
      </header>

      <div class="content-area">
        <!-- Dashboard -->
        <div v-if="activeSection === 'dashboard'" class="section">
          <div class="cards-grid">
            <div class="card">
              <h3>Total Usuarios</h3>
              <p class="card-number">1,234</p>
            </div>
            <div class="card">
              <h3>Ventas del Mes</h3>
              <p class="card-number">$45,678</p>
            </div>
            <div class="card">
              <h3>Productos</h3>
              <p class="card-number">567</p>
            </div>
            <div class="card">
              <h3>Pedidos Pendientes</h3>
              <p class="card-number">23</p>
            </div>
          </div>
        </div>

        <!-- Estudiantes -->
        <div v-if="activeSection === 'estudiantes'" class="section">
          <EstudiantesComponent />
        </div>

        <!-- Actividad -->
        <div v-if="activeSection === 'actividad'" class="section">
          <ActividadComponent />
        </div>

        <!-- Universidad -->
        <div v-if="activeSection === 'universidad'" class="section">
          <UniversidadComponent />
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import EstudiantesComponent from './EstudiantesComponent.vue'
import ActividadComponent from './ActividadComponent.vue'
import UniversidadComponent from './UniversidadComponent.vue'

export default {
  name: 'Dashboard',
  components: {
    EstudiantesComponent,
    ActividadComponent,
    UniversidadComponent
  },
  data() {
    return {
      sidebarCollapsed: false,
      activeSection: 'dashboard'
    }
  },
  methods: {
    toggleSidebar() {
      this.sidebarCollapsed = !this.sidebarCollapsed;
    },
    setActiveSection(section) {
      this.activeSection = section;
    },
    getSectionTitle() {
      const titles = {
        dashboard: 'Dashboard',
        estudiantes: 'Estudiantes',
        actividad: 'Actividad',
        universidad: 'Universidad'
      };
      return titles[this.activeSection] || 'Dashboard';
    },
    handleLogout() {
      // Emitir evento para notificar logout
      this.$emit('logout');
      
      // Opcional: redirect con router
      // this.$router.push('/login');
    }
  },
  mounted() {
    // Verificar si hay token al montar el componente
    // const token = localStorage.getItem('access_token');
    // if (!token) {
    //   this.$emit('logout');
    // }
  }
}
</script>

<style>
/* Reset global para pantalla completa */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body {
  height: 100%;
  overflow: hidden;
}

#app {
  height: 100vh;
  width: 100vw;
}
</style>

<style scoped>
.dashboard-container {
  display: flex;
  height: 100vh;
  width: 100vw;
  background-color: #f8f9fa;
  position: fixed;
  top: 0;
  left: 0;
  overflow: hidden;
}

/* Sidebar */
.sidebar {
  width: 250px;
  background-color: #2c3e50;
  color: white;
  transition: width 0.3s ease;
  display: flex;
  flex-direction: column;
  flex-shrink: 0;
  height: 100vh;
  overflow-y: auto;
}

.sidebar-collapsed {
  width: 70px;
}

.sidebar-header {
  padding: 1rem;
  border-bottom: 1px solid #34495e;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-shrink: 0;
}

.sidebar-header h3 {
  margin: 0;
  font-size: 1.2rem;
}

.toggle-btn {
  background: none;
  border: none;
  color: white;
  cursor: pointer;
  font-size: 1.2rem;
  padding: 0.25rem;
}

.sidebar-nav {
  flex: 1;
  padding: 1rem 0;
  overflow-y: auto;
}

.sidebar-nav ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.sidebar-nav li {
  margin-bottom: 0.5rem;
}

.sidebar-nav a {
  display: flex;
  align-items: center;
  padding: 0.75rem 1rem;
  color: #bdc3c7;
  text-decoration: none;
  transition: background-color 0.2s;
}

.sidebar-nav a:hover,
.sidebar-nav a.active {
  background-color: #34495e;
  color: white;
}

.sidebar-nav .icon {
  font-size: 1.2rem;
  width: 20px;
  text-align: center;
}

.sidebar-nav .text {
  margin-left: 0.75rem;
}

.sidebar-footer {
  padding: 1rem;
  border-top: 1px solid #34495e;
  flex-shrink: 0;
}

.logout-btn {
  display: flex;
  align-items: center;
  width: 100%;
  padding: 0.75rem;
  background: none;
  border: none;
  color: #e74c3c;
  cursor: pointer;
  border-radius: 4px;
  transition: background-color 0.2s;
}

.logout-btn:hover {
  background-color: rgba(231, 76, 60, 0.1);
}

.logout-btn .icon {
  font-size: 1.2rem;
  width: 20px;
  text-align: center;
}

.logout-btn .text {
  margin-left: 0.75rem;
}

/* Main Content */
.main-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  height: 100vh;
  overflow: hidden;
  min-width: 0;
}

.main-header {
  background: white;
  padding: 1rem 2rem;
  border-bottom: 1px solid #dee2e6;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-shrink: 0;
}

.main-header h1 {
  margin: 0;
  color: #2c3e50;
}

.user-info {
  color: #6c757d;
}

.content-area {
  flex: 1;
  padding: 2rem;
  overflow-y: auto;
  overflow-x: hidden;
}

/* Dashboard Cards */
.cards-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.card {
  background: white;
  padding: 1.5rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  text-align: center;
}

.card h3 {
  margin: 0 0 1rem 0;
  color: #6c757d;
  font-size: 0.9rem;
  text-transform: uppercase;
}

.card-number {
  font-size: 2rem;
  font-weight: bold;
  color: #2c3e50;
  margin: 0;
}

/* Sections */
.section {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  height: 100%;
  overflow-y: auto;
}

.form-group {
  margin-bottom: 1rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  color: #555;
  font-weight: 500;
}

.form-input {
  width: 100%;
  padding: 0.75rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
  max-width: 400px;
}

.setting-item {
  margin-bottom: 1rem;
}

.setting-item label {
  display: flex;
  align-items: center;
  cursor: pointer;
}

.setting-item input[type="checkbox"] {
  margin-right: 0.5rem;
}

.report-item {
  padding: 1rem;
  border: 1px solid #dee2e6;
  border-radius: 4px;
  margin-bottom: 1rem;
}

.btn-primary {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 0.5rem;
}

.btn-primary:hover {
  background-color: #0056b3;
}

/* Responsive */
@media (max-width: 768px) {
  .dashboard-container {
    flex-direction: column;
  }
  
  .sidebar {
    width: 100%;
    height: auto;
    position: relative;
    flex-shrink: 0;
  }
  
  .sidebar-collapsed {
    width: 100%;
    transform: none;
  }
  
  .main-content {
    height: calc(100vh - 60px);
  }
  
  .main-header {
    padding: 1rem;
  }
  
  .content-area {
    padding: 1rem;
  }
  
  .cards-grid {
    grid-template-columns: 1fr;
    gap: 1rem;
  }
}
</style>