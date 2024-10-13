<template>
  <div class="todo-app">
    <h2>ToDo List</h2>
    
    <div>
      <button @click="filterTasks('all')" :class="{ active: filter === 'all' }">Tất cả</button>
      <button @click="filterTasks('completed')" :class="{ active: filter === 'completed' }">Hoàn thành</button>
      <button @click="filterTasks('incomplete')" :class="{ active: filter === 'incomplete' }">Đang thực hiện</button>
    </div>

    <form @submit.prevent="addTask">
      <input v-model="newTask" placeholder="Enter your job" />
      <button type="submit">Add Job</button>
    </form>

    <ul>
      <li v-for="(task) in filteredTasks" :key="task.id">
        <input type="checkbox" v-model="task.status" @change="toggleTaskStatus(task)" />
        <span :class="{ completed: task.status }">{{ task.name }}</span>
        <button @click="editTask(task)">✏️</button>
        <button @click="deleteTask(task.id)">🗑️</button>
      </li>
    </ul>

    <div>
      Số công việc hoàn thành: {{ completedTasks }} / {{ tasks.length }} công việc
    </div>

    <button @click="deleteCompletedTasks">Xóa công việc hoàn thành</button>
    <button @click="deleteAllTasks">Xóa tất cả công việc</button>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
import axios from "axios";

const newTask = ref("");
const tasks = ref([]);
const filter = ref("all"); 

const completedTasks = computed(() => {
  return tasks.value.filter((task) => task.status).length;
});

const filteredTasks = computed(() => {
  if (filter.value === "completed") {
    return tasks.value.filter((task) => task.status);
  } else if (filter.value === "incomplete") {
    return tasks.value.filter((task) => !task.status);
  }
  return tasks.value;
});

const fetchTasks = async () => {
  try {
    const response = await axios.get(`http://localhost:3000/tasks`);
    tasks.value = response.data;
  } catch (error) {
    console.error("Error fetching tasks:", error);
  }
};

const addTask = async () => {
  if (!newTask.value) {
    alert("Công việc không được để trống");
    return;
  }
  const isDuplicate = tasks.value.some((task) => task.name === newTask.value);
  if (isDuplicate) {
    alert("Tên công việc không được phép trùng");
    return;
  }
  
  try {
    const task = {
      name: newTask.value,
      status: false,
    };
    const response = await axios.post(`http://localhost:3000/tasks`, task);
    tasks.value.push(response.data); 
    newTask.value = "";
  } catch (error) {
    console.error("Error adding task:", error);
  }
};

const editTask = (task) => {
  const updatedName = prompt("Nhập tên công việc mới:", task.name);
  if (updatedName) {
    task.name = updatedName;
    updateTask(task);
  }
};

const updateTask = async (task) => {
  try {
    await axios.put(`http://localhost:3000/tasks/${task.id}`, task);
  } catch (error) {
    console.error("Error updating task:", error);
  }
};

const deleteTask = async (taskId) => {
  if (confirm("Bạn có chắc chắn muốn xóa công việc này?")) {
    try {
      await axios.delete(`http://localhost:3000/tasks/${taskId}`);
      tasks.value = tasks.value.filter((task) => task.id !== taskId); 
    } catch (error) {
      console.error("Error deleting task:", error);
    }
  }
};

const toggleTaskStatus = async (task) => {
  try {
    await axios.put(`http://localhost:3000/tasks/${task.id}`, {
      ...task,
      status: task.status,
    });
  } catch (error) {
    console.error("Error updating task status:", error);
  }
};

const deleteCompletedTasks = async () => {
  if (confirm("Bạn có chắc chắn muốn xóa tất cả các công việc đã hoàn thành?")) {
    try {
      for (const task of tasks.value.filter((task) => task.status)) {
        await axios.delete(`http://localhost:3000/tasks/${task.id}`);
      }
      tasks.value = tasks.value.filter((task) => !task.status);
    } catch (error) {
      console.error("Error deleting completed tasks:", error);
    }
  }
};

const deleteAllTasks = async () => {
  if (confirm("Bạn có chắc chắn muốn xóa tất cả công việc?")) {
    try {
      for (const task of tasks.value) {
        await axios.delete(`http://localhost:3000/tasks/${task.id}`);
      }
      tasks.value = [];
    } catch (error) {
      console.error("Error deleting all tasks:", error);
    }
  }
};

const filterTasks = (type) => {
  filter.value = type;
};

onMounted(() => {
  fetchTasks();
});
</script>

<style scoped>
.todo-app {
  max-width: 500px;
  margin: auto;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}
.completed {
  text-decoration: line-through;
}
input[type="checkbox"] {
  margin-right: 10px;
  border-radius: 6px;
  padding: 5px 10px;
}

.active {
  background-color: blue;
  color: white;
}
</style>

