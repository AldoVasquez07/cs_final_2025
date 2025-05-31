<template>
  <div class="universidad-component">
    <div class="component-header">
      <h2>Gestión de Universidades</h2>
      <button @click="showCreateForm = true" class="btn btn-primary">
        Nueva Universidad
      </button>
    </div>

    <!-- Formulario de creación/edición -->
    <div v-if="showCreateForm || editingItem" class="form-modal">
      <div class="form-overlay" @click="cancelForm"></div>
      <div class="form-content">
        <h3>{{ editingItem ? 'Editar Universidad' : 'Nueva Universidad' }}</h3>
        <form @submit.prevent="submitForm">
          <div class="form-group">
            <label for="nombre">Nombre *</label>
            <input
              id="nombre"
              v-model="formData.nombre"
              type="text"
              class="form-input"
              required
              placeholder="Ingrese el nombre de la universidad"
            />
          </div>

          <div class="form-actions">
            <button type="button" @click="cancelForm" class="btn btn-secondary">
              Cancelar
            </button>
            <button type="submit" class="btn btn-primary" :disabled="loading">
              {{ loading ? 'Guardando...' : (editingItem ? 'Actualizar' : 'Crear') }}
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Información de paginación y búsqueda -->
    <div class="table-controls">
      <div class="search-container">
        <input
          v-model="searchTerm"
          type="text"
          placeholder="Buscar universidades..."
          class="search-input"
          @input="handleSearch"
        />
      </div>
      <div class="pagination-info">
        Mostrando {{ startIndex + 1 }}-{{ endIndex }} de {{ totalItems }} registros
      </div>
    </div>

    <!-- Lista de universidades -->
    <div class="data-table">
      <div v-if="loading && !universidades.length" class="loading">
        Cargando universidades...
      </div>
      
      <div v-else-if="error" class="error">
        {{ error }}
        <button @click="fetchUniversidades" class="btn btn-secondary">Reintentar</button>
      </div>
      
      <div v-else-if="!paginatedUniversidades.length" class="empty-state">
        {{ searchTerm ? 'No se encontraron universidades que coincidan con la búsqueda' : 'No hay universidades registradas' }}
      </div>
      
      <div v-else class="table-wrapper">
        <table class="table">
          <thead>
            <tr>
              <th>ID</th>
              <th>Nombre</th>
              <th class="actions-column">Acciones</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="universidad in paginatedUniversidades" :key="universidad.id">
              <td class="text-content id-column">{{ universidad.id }}</td>
              <td class="text-content nombre-column">{{ universidad.nombre }}</td>
              <td class="actions-column">
                <div class="action-buttons">
                  <button @click="editItem(universidad)" class="btn btn-sm btn-warning">
                    Editar
                  </button>
                  <button @click="deleteItem(universidad)" class="btn btn-sm btn-danger">
                    Eliminar
                  </button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Paginación -->
    <div v-if="totalPages > 1" class="pagination-container">
      <div class="pagination">
        <button 
          @click="goToPage(1)" 
          :disabled="currentPage === 1"
          class="btn btn-pagination"
        >
          ‹‹
        </button>
        <button 
          @click="goToPage(currentPage - 1)" 
          :disabled="currentPage === 1"
          class="btn btn-pagination"
        >
          ‹
        </button>
        
        <template v-for="page in visiblePages" :key="page">
          <button 
            v-if="page !== '...'"
            @click="goToPage(page)"
            :class="['btn', 'btn-pagination', { 'active': page === currentPage }]"
          >
            {{ page }}
          </button>
          <span v-else class="pagination-dots">...</span>
        </template>
        
        <button 
          @click="goToPage(currentPage + 1)" 
          :disabled="currentPage === totalPages"
          class="btn btn-pagination"
        >
          ›
        </button>
        <button 
          @click="goToPage(totalPages)" 
          :disabled="currentPage === totalPages"
          class="btn btn-pagination"
        >
          ››
        </button>
      </div>
      
      <div class="pagination-controls">
        <select v-model="itemsPerPage" @change="changeItemsPerPage" class="items-per-page-select">
          <option value="5">5 por página</option>
          <option value="10">10 por página</option>
          <option value="20">20 por página</option>
          <option value="50">50 por página</option>
        </select>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'UniversidadComponent',
  data() {
    return {
      universidades: [],
      loading: false,
      error: null,
      showCreateForm: false,
      editingItem: null,
      formData: {
        nombre: ''
      },
      // Paginación
      currentPage: 1,
      itemsPerPage: 10,
      // Búsqueda
      searchTerm: '',
      filteredUniversidades: []
    }
  },
  computed: {
    totalItems() {
      return this.filteredUniversidades.length;
    },
    totalPages() {
      return Math.ceil(this.totalItems / this.itemsPerPage);
    },
    startIndex() {
      return (this.currentPage - 1) * this.itemsPerPage;
    },
    endIndex() {
      return Math.min(this.startIndex + this.itemsPerPage, this.totalItems);
    },
    paginatedUniversidades() {
      return this.filteredUniversidades.slice(this.startIndex, this.endIndex);
    },
    visiblePages() {
      const pages = [];
      const maxVisible = 5;
      
      if (this.totalPages <= maxVisible) {
        for (let i = 1; i <= this.totalPages; i++) {
          pages.push(i);
        }
      } else {
        const start = Math.max(1, this.currentPage - 2);
        const end = Math.min(this.totalPages, this.currentPage + 2);
        
        if (start > 1) {
          pages.push(1);
          if (start > 2) pages.push('...');
        }
        
        for (let i = start; i <= end; i++) {
          pages.push(i);
        }
        
        if (end < this.totalPages) {
          if (end < this.totalPages - 1) pages.push('...');
          pages.push(this.totalPages);
        }
      }
      
      return pages;
    }
  },
  watch: {
    universidades() {
      this.applyFilters();
    }
  },
  mounted() {
    this.fetchUniversidades();
  },
  methods: {
    async fetchUniversidades() {
      this.loading = true;
      this.error = null;
      
      try {
        const token = localStorage.getItem('access_token');
        const response = await fetch('http://localhost:8001/silabo/universidades/', {
          headers: {
            'Authorization': `Bearer ${token}`,
            'Content-Type': 'application/json'
          }
        });
        
        if (!response.ok) {
          throw new Error('Error al cargar las universidades');
        }
        
        this.universidades = await response.json();
      } catch (error) {
        this.error = error.message;
        console.error('Error:', error);
      } finally {
        this.loading = false;
      }
    },

    async submitForm() {
      this.loading = true;
      this.error = null;
      
      try {
        const token = localStorage.getItem('access_token');
        const url = this.editingItem 
          ? `http://localhost:8001/silabo/universidades/${this.editingItem.id}/`
          : 'http://localhost:8001/silabo/universidades/';
        
        const method = this.editingItem ? 'PUT' : 'POST';
        
        const response = await fetch(url, {
          method,
          headers: {
            'Authorization': `Bearer ${token}`,
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(this.formData)
        });
        
        if (!response.ok) {
          const errorData = await response.json();
          throw new Error(Object.values(errorData).flat().join(', '));
        }
        
        await this.fetchUniversidades();
        this.cancelForm();
      } catch (error) {
        this.error = error.message;
        console.error('Error:', error);
      } finally {
        this.loading = false;
      }
    },

    async deleteItem(universidad) {
      if (!confirm(`¿Está seguro de eliminar la universidad "${universidad.nombre}"?`)) {
        return;
      }
      
      this.loading = true;
      this.error = null;
      
      try {
        const token = localStorage.getItem('access_token');
        const response = await fetch(`http://localhost:8001/silabo/universidades/${universidad.id}/`, {
          method: 'DELETE',
          headers: {
            'Authorization': `Bearer ${token}`,
            'Content-Type': 'application/json'
          }
        });
        
        if (!response.ok) {
          throw new Error('Error al eliminar la universidad');
        }
        
        await this.fetchUniversidades();
      } catch (error) {
        this.error = error.message;
        console.error('Error:', error);
      } finally {
        this.loading = false;
      }
    },

    editItem(universidad) {
      this.editingItem = universidad;
      this.formData = {
        nombre: universidad.nombre
      };
    },

    cancelForm() {
      this.showCreateForm = false;
      this.editingItem = null;
      this.formData = {
        nombre: ''
      };
      this.error = null;
    },

    // Métodos de paginación
    goToPage(page) {
      if (page >= 1 && page <= this.totalPages) {
        this.currentPage = page;
      }
    },

    changeItemsPerPage() {
      this.currentPage = 1;
    },

    // Métodos de búsqueda
    handleSearch() {
      this.currentPage = 1;
      this.applyFilters();
    },

    applyFilters() {
      if (!this.searchTerm.trim()) {
        this.filteredUniversidades = [...this.universidades];
      } else {
        const term = this.searchTerm.toLowerCase();
        this.filteredUniversidades = this.universidades.filter(universidad =>
          universidad.nombre.toLowerCase().includes(term) ||
          universidad.id.toString().includes(term)
        );
      }
    }
  }
}
</script>

<style scoped>
.universidad-component {
  height: 100%;
  display: flex;
  flex-direction: column;
  background-color: #f8f9fa;
  padding: 1rem;
}

.component-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid #dee2e6;
  background: white;
  padding: 1.5rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.component-header h2 {
  margin: 0;
  color: #2c3e50;
  font-size: 1.5rem;
  font-weight: 600;
}

.btn {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: all 0.2s;
  font-weight: 500;
}

.btn-primary {
  background-color: #007bff;
  color: white;
}

.btn-primary:hover:not(:disabled) {
  background-color: #0056b3;
  transform: translateY(-1px);
}

.btn-secondary {
  background-color: #6c757d;
  color: white;
}

.btn-secondary:hover {
  background-color: #545b62;
}

.btn-warning {
  background-color: #ffc107;
  color: #212529;
}

.btn-warning:hover {
  background-color: #e0a800;
}

.btn-danger {
  background-color: #dc3545;
  color: white;
}

.btn-danger:hover {
  background-color: #c82333;
}

.btn-sm {
  padding: 0.25rem 0.5rem;
  font-size: 0.8rem;
}

.btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.form-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
}

.form-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
}

.form-content {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  width: 90%;
  max-width: 500px;
  max-height: 90vh;
  overflow-y: auto;
  position: relative;
  z-index: 1001;
}

.form-content h3 {
  margin: 0 0 1.5rem 0;
  color: #2c3e50;
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
  font-family: inherit;
  color: #495057;
  background-color: white;
}

.form-input:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.25);
}

.form-actions {
  display: flex;
  gap: 1rem;
  justify-content: flex-end;
  margin-top: 1.5rem;
}

/* Controles de tabla */
.table-controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  background: white;
  padding: 1rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.search-container {
  flex: 1;
  max-width: 300px;
}

.search-input {
  width: 100%;
  padding: 0.5rem 1rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 0.9rem;
  color: #495057;
  background-color: white;
}

.search-input:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.15);
}

.pagination-info {
  color: #6c757d;
  font-size: 0.9rem;
}

.data-table {
  flex: 1;
  display: flex;
  flex-direction: column;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  min-height: 0;
}

.table-wrapper {
  flex: 1;
  overflow: auto;
  min-height: 0;
}

.loading, .error, .empty-state {
  text-align: center;
  padding: 2rem;
  color: #6c757d;
  background: white;
}

.error {
  color: #dc3545;
}

.table {
  width: 100%;
  border-collapse: collapse;
  background: white;
  min-width: 600px;
}

.table th,
.table td {
  padding: 1rem;
  text-align: left;
  border-bottom: 1px solid #dee2e6;
  white-space: nowrap;
}

.table th {
  background-color: #f8f9fa;
  font-weight: 600;
  color: #495057;
  position: sticky;
  top: 0;
  z-index: 10;
}

.table tbody tr:hover {
  background-color: #f8f9fa;
}

.text-content {
  color: #212529 !important;
  font-size: 0.9rem;
}

/* Columnas específicas */
.id-column {
  width: 80px;
  min-width: 80px;
}

.nombre-column {
  width: auto;
  min-width: 300px;
  white-space: normal;
}

.actions-column {
  width: 160px;
  min-width: 160px;
}

.action-buttons {
  display: flex;
  gap: 0.5rem;
  flex-wrap: nowrap;
}

/* Paginación */
.pagination-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 1rem;
  background: white;
  padding: 1rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.pagination {
  display: flex;
  gap: 0.25rem;
  align-items: center;
}

.btn-pagination {
  padding: 0.5rem 0.75rem;
  background: white;
  border: 1px solid #dee2e6;
  color: #007bff;
  font-size: 0.9rem;
  min-width: 40px;
}

.btn-pagination:hover:not(:disabled) {
  background-color: #e9ecef;
  border-color: #adb5bd;
}

.btn-pagination:disabled {
  color: #6c757d;
  background-color: #f8f9fa;
  border-color: #dee2e6;
}

.btn-pagination.active {
  background-color: #007bff;
  border-color: #007bff;
  color: white;
}

.pagination-dots {
  padding: 0.5rem;
  color: #6c757d;
}

.pagination-controls {
  display: flex;
  align-items: center;
}

.items-per-page-select {
  padding: 0.4rem 0.75rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 0.9rem;
  color: #495057;
  background-color: white;
  cursor: pointer;
}

.items-per-page-select:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.15);
}

@media (max-width: 768px) {
  .universidad-component {
    padding: 0.5rem;
  }
  
  .component-header {
    flex-direction: column;
    gap: 1rem;
    text-align: center;
  }
  
  .table-controls {
    flex-direction: column;
    gap: 1rem;
    align-items: stretch;
  }
  
  .search-container {
    max-width: none;
  }
  
  .form-content {
    width: 95%;
    padding: 1.5rem;
  }
  
  .table {
    font-size: 0.8rem;
    min-width: 500px;
  }
  
  .table th,
  .table td {
    padding: 0.5rem;
  }
  
  .nombre-column {
    min-width: 200px;
  }
  
  .actions-column {
    width: 140px;
    min-width: 140px;
  }
  
  .action-buttons {
    flex-direction: column;
    gap: 0.25rem;
  }
  
  .pagination-container {
    flex-direction: column;
    gap: 1rem;
  }
  
  .pagination {
    justify-content: center;
    flex-wrap: wrap;
  }
}
</style>