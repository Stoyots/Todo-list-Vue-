<template>
  <div id="app">
    <!-- Sidebar -->
    <div class="sidebar" :class="{ collapsed: isSidebarCollapsed }">
      <div class="sidebar-header">
        <h2 v-if="!isSidebarCollapsed">My Lists</h2>
        <!-- Toggle button next to the title -->
        <button @click="toggleSidebar" class="toggle-sidebar-button">
          {{ isSidebarCollapsed ? '>' : '<' }}
        </button>
      </div>
      
      <ul v-if="!isSidebarCollapsed">
        <li 
          v-for="(list, index) in lists" 
          :key="index" 
          :class="{ active: index === currentListIndex }"
          @click="selectList(index)"
          :style="{ backgroundColor: list.color }"
        >
          <span class="color-swatch" :style="{ backgroundColor: list.color }"></span>
          {{ list.name }}
        </li>
      </ul>
      <button v-if="!isSidebarCollapsed" @click="addNewList" class="add-list-button">+ Add List</button>
    </div>

    <!-- Main area -->
    <div class="main" :style="{ backgroundColor: currentList.color }">
      <div class="list-header">
        <template v-if="isEditingListName">
          <input 
            type="text" 
            v-model="currentList.name" 
            @blur="stopEditingListName" 
            @keyup.enter="stopEditingListName"
            class="list-name-input"
          />
        </template>
        <template v-else>
          <h1 @click="startEditingListName">{{ currentList.name }}</h1>
        </template>
        
        <!-- Custom color picker -->
        <div class="color-picker">
          <button
            v-for="(hex, name) in colorNames"
            :key="name"
            :style="{ backgroundColor: hex }"
            class="color-option"
            @click="selectColor(hex)"
          >
            <!-- Small colored square as preview -->
          </button>
        </div>
      </div>

      <div class="task-manager">
        <input 
          type="text" 
          v-model="newTask" 
          @keyup.enter="addTask" 
          placeholder="Add a task..."
        />
        
        <!-- Active tasks -->
        <ul>
          <li 
            v-for="(task, index) in activeTasks" 
            :key="task.id"
          >
            <input 
              type="checkbox" 
              v-model="task.completed" 
              @change="saveLists"
            />
            <span>{{ task.name }}</span>
            <button @click="deleteTask(index)">Delete</button>
          </li>
        </ul>

        <!-- Separator -->
        <hr />

        <!-- Completed tasks -->
        <ul>
          <li 
            v-for="(task, index) in completedTasks" 
            :key="task.id"
          >
            <input 
              type="checkbox" 
              v-model="task.completed" 
              @change="saveLists"
            />
            <span class="completed">{{ task.name }}</span>
            <button @click="deleteTask(index)">Delete</button>
          </li>
        </ul>

        <!-- Delete list button -->
        <button @click="deleteList(currentListIndex)" class="delete-list-button">
          Delete List
        </button>
      </div>
    </div>
  </div>
</template>


<script>
export default {
  data() {
    return {
      colorNames: {
        "Rose Clair": "#FAD0C9",      // Rose pastel doux
        "Jaune Doré": "#F9E79F",      // Jaune pastel
        "Bleu Nuit": "#A3BFD9",       // Bleu pastel clair
        "Vert Clair": "#A8E6CF",      // Vert pastel doux
        "Tomate": "#FAD0B1",          // Rouge pastel doux
        "Lavande": "#D7B6D4",         // Lavande pastel clair
        "Jaune Pâle": "#F4E1A1",      // Jaune pâle doux
        "Bleu Turquoise": "#B2D8D8",  // Bleu turquoise pastel
        "Vert Forêt": "#C8E6C9", 
      },
      lists: JSON.parse(localStorage.getItem("lists")) || [
        { name: "Ma première liste", tasks: [], color: "#FFB6C1" },
      ],
      currentListIndex: 0,
      newTask: "",
      isEditingListName: false,
      isSidebarCollapsed: false,
    };
  },
  computed: {
    currentList() {
      return this.lists[this.currentListIndex];
    },
    activeTasks() {
      return this.currentList.tasks.filter((task) => !task.completed);
    },
    completedTasks() {
      return this.currentList.tasks.filter((task) => task.completed);
    },
  },
  methods: {
    addNewList() {
      const colorsArray = Object.values(this.colorNames);
      const newList = {
        name: `New List ${this.lists.length + 1}`,
        tasks: [],
        color: colorsArray[Math.floor(Math.random() * colorsArray.length)],
      };
      this.lists.push(newList);
      this.currentListIndex = this.lists.length - 1;
      this.saveLists();
    },
    selectList(index) {
      this.currentListIndex = index;
    },
    addTask() {
      if (this.newTask.trim()) {
        this.currentList.tasks.push({
          id: Date.now(),
          name: this.newTask,
          completed: false,
        });
        this.newTask = "";
        this.saveLists();
      }
    },
    deleteTask(index) {
      this.currentList.tasks.splice(index, 1);
      this.saveLists();
    },
    startEditingListName() {
      this.isEditingListName = true;
    },
    stopEditingListName() {
      this.isEditingListName = false;
      this.saveLists();
    },
    deleteList(index) {
      if (this.lists.length === 1) {
        // Si c'est la dernière liste, on en crée une nouvelle vide
        this.lists.splice(0, 1, { name: "New List", tasks: [], color: "#FFB6C1" });
      } else {
        if (confirm("Êtes-vous sûr de vouloir supprimer cette liste ?")) {
          this.lists.splice(index, 1);
        }
      }
      // Si on supprime une liste, on met à jour l'index de la liste sélectionnée
      if (this.currentListIndex >= this.lists.length) {
        this.currentListIndex = this.lists.length - 1;
      }
      this.saveLists();
    },
    selectColor(hex) {
      this.currentList.color = hex;
      this.saveLists();
    },
    saveLists() {
      localStorage.setItem("lists", JSON.stringify(this.lists));
    },
    toggleSidebar() {
      this.isSidebarCollapsed = !this.isSidebarCollapsed;
    },
  },
};
</script>

<style scoped>
#app {
  display: flex;
  height: 100vh;
  font-family: 'Arial', sans-serif;
  background-color: #f7f7f7;
}

.sidebar {
  width: 170px;
  background-color: #949494a8;
  padding: 20px;
  border-right: 1px solid #e1e1e1;
  box-shadow: 4px 0 10px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  margin-right: 20px;
  transition: width 0.3s ease;
}

.sidebar.collapsed {
  width: 50px;
}

.sidebar-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.sidebar h2 {
  margin-top: 0;
  font-size: 1.5rem;
  color: #333;
}

.sidebar ul {
  list-style: none;
  padding: 0;
}

.sidebar li {
  padding: 12px;
  margin-bottom: 10px;
  cursor: pointer;
  font-size: 1.2rem;
  border-radius: 7px;
  transition: background-color 0.3s, transform 0.2s;
}

.sidebar li:hover {
  background-color: #f1f1f1;
  transform: translateX(10px);
}

.sidebar li.active {
  background-color: #4CAF50;
  color: white;
  font-weight: bold;
  transform: translateX(10px);
}

.sidebar .add-list-button {
  margin-top: 20px;
  padding: 12px;
  width: 100%;
  background-color: #5e5e5e;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1.2rem;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  transition: background-color 0.3s, transform 0.2s;
}

.sidebar .add-list-button:hover {
  background-color: #218838;
  transform: scale(1.05);
}

.sidebar .toggle-sidebar-button {
  background-color: #5e5e5e;
  color: white;
  border: none;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  cursor: pointer;
  font-size: 1.5rem;
  text-align: center;
  margin-left: 20px;
}

.main {
  flex: 1;
  padding: 50px;
  border-radius: 8px;
  background-color: #ffffff;
  margin: 10vh;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.425);
  position: relative;
  transition: 0.3s;
}

.main:hover{
  box-shadow: rgba(0, 0, 0, 0.25) 0px 54px 55px, rgba(0, 0, 0, 0.12) 0px -12px 30px, rgba(0, 0, 0, 0.12) 0px 4px 6px, rgba(0, 0, 0, 0.17) 0px 12px 13px, rgba(0, 0, 0, 0.09) 0px -3px 5px;
  scale: 1.07;
  transition: 0.3s;
}

.list-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.list-header h1 {
  margin: 0;
  font-size: 2rem;
  color: #333;
  cursor: pointer;
  transition: color 0.3s;
}

.list-header h1:hover {
  color: #4CAF50;
}

.color-picker {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  width: 100px;
}

.color-option {
  width: 20px;
  height: 20px;
  border-radius: 5px;
  border: 1px solid #27272779;
  cursor: pointer;
}

.task-manager {
  margin-top: 20px;
}

.task-manager input[type="text"] {
  width: 40%;
  padding: 10px;
  font-size: 1rem;
  border-radius: 8px;
  border: 1px solid #ddd;
  margin-bottom: 13px;
  transition: border-color 0.7s;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.05);
}

.task-manager input[type="text"]:focus {
  border-color: #242424;
  outline: none;
}

.task-manager ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.task-manager li {
  display: flex;
  align-items: center;
  background-color: #f9f9f952;
  margin-bottom: 12px;
  padding: 3px;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.05);
  transition: transform 0.3s, background-color 0.3s;
}

.task-manager li:hover {
  background-color: #f1f1f1;
  transform: translateX(5px);
}

.task-manager li .completed {
  text-decoration: line-through;
  color: #bbb;
}

.task-manager li button {
  background-color: #FF6347;
  border: none;
  color: white;
  padding: 6px 12px;
  border-radius: 5px;
  cursor: pointer;
  margin-left: auto;
  transition: background-color 0.3s, transform 0.2s;
}

.task-manager li button:hover {
  background-color: #d9534f;
  transform: scale(1.05);
}

.delete-list-button {
  position: absolute;
  bottom: 20px;
  right: 20px;
  padding: 8px 16px;
  background-color: #FF6347;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.7rem;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  transition: background-color 0.3s, transform 0.2s;
}

.delete-list-button:hover {
  background-color: #d9534f;
  transform: scale(1.05);
}
</style>
