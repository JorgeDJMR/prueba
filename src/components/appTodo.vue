<template>
  <div class="container mt-5">
    <form @submit.prevent="handleSubmit">
      <div class="mb-3">
        <input 
          v-model="taskTitle" 
          type="text" 
          class="form-control" 
          id="taskTitle" 
          placeholder="Ingresa el título" 
          maxlength="16"
          required
        >
      </div>
      <div class="mb-3">
        <textarea 
          v-model="taskC" 
          class="form-control" 
          id="taskC" 
          placeholder="Ingresa tu tarea" 
          rows="4"
          required
        ></textarea>
      </div>
      <div class="ml-3 d-flex flex-column">
        <input 
          v-model="taskDate" 
          type="date" 
          class="form-control small-date" 
          id="taskDate" 
          required
        >
        <div v-if="dateError" class="alert alert-danger mt-2" role="alert">
          La fecha debe tener un año de exactamente 4 dígitos y tiene que ser una fecha futura.
        </div>
      </div>
      <button type="submit" class="btn btn-primary" style="padding: 10px 10px">{{ isEditing ? 'Actualizar tarea' : 'Agregar tarea' }}</button>
      <button 
        v-if="isEditing" 
        type="button" 
        class="btn btn-secondary cancel-edit-button " 
        style="padding: 10px 10px"
        @click="cancelEdit"
      >Cancelar edición</button>
    </form>

    <!-- Notificación de éxito -->
    <div v-if="showSuccess" class="alert alert-success mt-3" role="alert">
      ¡Tarea añadida con éxito!
    </div>

    <h2 class="mt-5">Tus tareas</h2>
    <div class="btn-group mb-3" role="group">
      <button 
        type="button" 
        :class="['btn-filter', filter === 'pending' ? 'btn-filter-active' : '']"
        @click="filter = 'pending'"
      >Pendientes</button>
      <button 
        type="button" 
        :class="['btn-filter', filter === 'completed' ? 'btn-filter-active' : '']"
        @click="filter = 'completed'"
      >Completadas</button>
      <button 
        type="button" 
        :class="['btn-filter', filter === 'all' ? 'btn-filter-active' : '']"
        @click="filter = 'all'"
      >Todas</button>
      <!-- Botón para alternar entre 1 fila y 3 filas -->
      <button 
        type="button" 
        class="btn btn-secondary toggle-view-button" 
        @click="toggleView"
      >
        <img src="@/assets/menu.png" alt="Toggle View" class="icon">
      </button>
    </div>
    
    <div class="row" :class="{ 'single-column': singleColumn }" id="todoList">
      <div 
        v-for="todo in filteredTodos" 
        :key="todo.id" 
        :class="['col-md-4', { 'col-md-12': singleColumn }]"
      >
        <div 
          class="card mb-3" 
          :class="{ 'bg-success': todo.completed, 'selected-card': todo.isSelected }"
        >
          <div class="card-body">
            <div class="icon-container-title">
              <img 
                src="@/assets/mitachuela2.png" 
                alt="Pin" 
                class="icon pin-icon" 
                @click="selectTodoItem(todo.id)"
                :class="{ 'disabled-icon': isEditing }"
              />
              <h5 class="card-title">{{ todo.title }}</h5>
            </div>
            <h6 class="card-text">{{ todo.name }}</h6>
            <p class="card-date">Fecha de inicio: {{ todo.date }}</p>
            <div class="icon-container">
              <img 
                src="@/assets/completo3.png" 
                alt="Complete" 
                class="icon" 
                @click="toggleTodoComplete(todo.id)"
                :class="{ 'disabled-icon': isEditing }"
              />
              <img 
                src="@/assets/editar2.png" 
                alt="Edit" 
                class="icon" 
                @click="editTodoItem(todo.id)"
                :class="{ 'disabled-icon': isEditing }"
              />
              <img 
                src="@/assets/borrar2.png" 
                alt="Delete" 
                class="icon" 
                @click="deleteTodoItem(todo.id)"
                :class="{ 'disabled-icon': isEditing }"
              />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      taskTitle: '',
      taskC: '',
      taskDate: '',
      todos: [],
      isEditing: false,
      editingIndex: null,
      filter: 'pending', // Estado inicial del filtro
      singleColumn: false, // Estado para alternar entre 1 fila y 3 filas
      showSuccess: false, // Estado para mostrar la notificación de éxito
      dateError: false, // Estado para mostrar el error de fecha
    };
  },
  mounted() {
    this.loadTodos();
  },
  computed: {
    filteredTodos() {
      if (this.filter === 'completed') {
        return this.todos.filter(todo => todo.completed);
      } else if (this.filter === 'all') {
        return this.todos;
      }
      return this.todos.filter(todo => !todo.completed);
    }
  },
  methods: {
    handleSubmit() {
      if (this.isValidDate(this.taskDate)) {
        this.dateError = false;
        if (this.isEditing) {
          this.updateTodoItem();
        } else {
          this.addTodoItem();
          this.showSuccessNotification();
        }
        this.taskTitle = '';
        this.taskC = '';
        this.taskDate = '';
        this.isEditing = false;
        this.editingIndex = null;
      } else {
        this.dateError = true;
      }
    },
    isValidDate(dateString) {
      const [year] = dateString.split('-');
      const date = new Date(dateString);
      const currentDate = new Date();
  
      // Verifica que el año tenga 4 dígitos y la fecha no sea pasada
      return year.length === 4 && date.getTime() >= currentDate.setHours(0, 0, 0, 0);
     },
    addTodoItem() {
      const newTodo = {
        id: Date.now(), // Usar un timestamp como ID único
        title: this.taskTitle,
        name: this.taskC,
        date: this.taskDate,
        completed: false,
        isSelected: false,
      };
      this.todos.unshift(newTodo);
      this.saveTodos();
    },
    deleteTodoItem(id) {
      if (!this.isEditing) {
        this.todos = this.todos.filter(todo => todo.id !== id);
        this.saveTodos();
      }
    },
    toggleTodoComplete(id) {
      if (!this.isEditing) {
        const todo = this.todos.find(todo => todo.id === id);
        if (todo) {
          todo.completed = !todo.completed;
          this.saveTodos();
        }
      }
    },
    editTodoItem(id) {
      if (!this.isEditing) {
        const todo = this.todos.find(todo => todo.id === id);
        if (todo) {
          this.taskTitle = todo.title;
          this.taskC = todo.name;
          this.taskDate = todo.date;
          this.isEditing = true;
          this.editingIndex = this.todos.indexOf(todo);

          this.$nextTick(() => {
            window.scrollTo({
              top: 0,
              behavior: 'smooth'
            });
          });
        }
      }
    },
    selectTodoItem(id) {
      if (!this.isEditing) {
        const todo = this.todos.find(todo => todo.id === id);
        if (todo) {
          // Cambiar el estado de selección
          todo.isSelected = !todo.isSelected;
 
          // Mover el ítem seleccionado al principio
          if (todo.isSelected) {
            this.todos = this.todos.filter(t => t.id !== id); // Eliminar de su posición actual
            this.todos.unshift(todo); // Añadir al principio
          } else {
            // Si se deselecciona, moverlo al final
            this.todos = this.todos.filter(t => t.id !== id); // Eliminar de su posición actual
            this.todos.push(todo); // Añadir al final
          }
 
          this.saveTodos();
        }
      }
    },
    updateTodoItem() {
      const updatedTodo = {
        ...this.todos[this.editingIndex],
        title: this.taskTitle,
        name: this.taskC,
        date: this.taskDate,
      };
      this.todos.splice(this.editingIndex, 1, updatedTodo);
      this.saveTodos();
    },
    cancelEdit() {
      this.isEditing = false;
      this.taskTitle = '';
      this.taskC = '';
      this.taskDate = '';
    },
    toggleView() {
      this.singleColumn = !this.singleColumn;
    },
    showSuccessNotification() {
      this.showSuccess = true;
      setTimeout(() => {
        this.showSuccess = false;
      }, 3000); // Ocultar notificación después de 3 segundos
    },
    loadTodos() {
      const savedTodos = JSON.parse(localStorage.getItem('todos')) || [];
      this.todos = savedTodos;
    },
    saveTodos() {
      localStorage.setItem('todos', JSON.stringify(this.todos));
    }
  }
};
</script>


<style scoped>
.small-date {
  width: 150px; /* Ajusta el ancho según sea necesario */
  margin:0px 0px 15px 0px
}

.d-flex {
  display: flex;

}

.card-text {
  height: 6.5rem;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 3; /* Número máximo de líneas que se mostrarán */
  -webkit-box-orient: vertical;
}

.logoedit {
  width: 100px; 
  height: auto; /* Mantén la proporción original */
}

#todoForm {
  max-width: 600px;
  margin: auto;
}

.form-label{
  color: #000000;
}

#todoList {
  margin-top: 20px;
}

.card-text {
  height: 300px; /* Altura fija para el área de texto */
  overflow-y: auto; /* Muestra la barra de desplazamiento solo cuando sea necesario */
}

h2{
  color: #000000;
}

textarea {
  resize: none;
  min-height: 100px;
}

.cancel-edit-button {
  margin-left: 30px;
}

.icon-container {
  display: flex;
  gap: 10px;
}

.icon {
  width: 30px;
  height: 30px;
  cursor: pointer;
  transition: transform 0.2s;
}

.icon:hover {
  transform: scale(1.2);
}

.icon-container-title {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 20px;
  
}

.pin-icon {
  position: absolute;
  top: 0;
  left: 0;
  width: 30px;
  height: 30px;
  cursor: pointer;
}

.card-title {
  text-align: center;
  font-size: 1.2rem;
  font-weight: bold;
  width: calc(100% - 30px); /* Ajustar el ancho para dejar espacio para el icono */
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  
}

.selected-card {
  border: 3px solid blue;
}

.card {
  position: relative;
}

.disabled-icon {
  opacity: 0.5;
  cursor: not-allowed;
  pointer-events: none;
}

/* Estilo predeterminado para los botones de filtro */
.btn-filter {
  background-color: #f0f0f0; /* Gris claro */
  color: #333; /* Color de texto oscuro para contraste */
  border: 1px solid #ccc; /* Borde gris claro */
  border-radius: 5px; /* Bordes redondeados */
  padding: 10px 20px; /* Espaciado interno */
  margin: 0 5px; /* Espaciado entre botones */
  transition: background-color 0.3s, box-shadow 0.3s; /* Transición suave */
  
}

.btn-filter:hover {
  background-color: #e0e0e0; /* Gris ligeramente más oscuro en hover */
}

/* Estilo para el botón seleccionado */
.btn-filter-active {
  background-color: #007bff; /* Azul para el botón activo */
  color: white; /* Texto blanco en el botón activo */
  border: 1px solid #007bff; /* Borde azul para el botón activo */
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1); /* Sombra para efecto de elevación */
}

/* Otros estilos de botones */
.btn-info {
  background-color: #17a2b8;
  color: white;
}

.btn-success {
  background-color: #28a745;
  color: white;
}

.btn-secondary {
  background-color: #6c757d;
  color: white;
}

.card-date {
  font-size: 0.9rem;
}


/* Estilo para el botón de alternar entre 1 fila y 3 filas */
.toggle-view-button {
  background: transparent; /* Fondo transparente */
  margin-left: 10px !important; /* Separación de 10 px a la izquierda, con !important */
  border: none; /* Sin borde */
  padding: 0; /* Sin espaciado interno */
  cursor: pointer; /* Cambia el cursor a puntero */
  display: inline-block; /* Asegura que el margen funcione correctamente */
}

.toggle-view-button:hover,
.toggle-view-button:focus {
  background: transparent; /* Sin color de fondo en hover/focus */
}

/* Estilo para el botón de alternar entre 1 fila y 3 filas */

/* Media query para manejar el espacio en pantallas medianas */
@media (max-width: 993px) and (min-width: 768px) {
  .icon-container-title {
    padding-left: 30px; /* Aumentar el espacio para el icono */
  }
  .card-title {
    width: calc(100% - 30px); /* Ajustar el ancho para el nuevo espacio del icono */
  }
}

/* Media query para manejar el tamaño de los botones en pantallas menores a 500px */
@media (max-width: 450px) {
  .btn-filter, .toggle-view-button {
    padding: 5px 15px; /* Reducir el tamaño del padding en 5px */
    font-size: 0.875rem; /* Opcional: ajustar el tamaño de la fuente si es necesario */
  }
  .toggle-view-button {
  margin-left: 0px !important; /* Separación de 10 px a la izquierda, con !important */
  }
}
</style>
