<script lang="ts">
import { computed, onMounted, ref, watch } from 'vue'
type Task = {
  id: number
  name: string
  priority: 'High' | 'Medium' | 'Low'
  done: boolean
}

export default {
  setup() {
    //! refs
    const newTaskInput = ref<HTMLInputElement | null>(null)
    const newTask = ref<string>('')
    const priority = ref<string>('Low')
    const errorMessage = ref<string | null>(null)
    const tasks = ref<Task[]>([])
    const filter = ref<string>('')
    //! lifecyle hooks
    onMounted(() => {
      newTaskInput.value?.focus()
      const storedTask = localStorage.getItem('tasks')
      if (!storedTask) return
      tasks.value = JSON.parse(storedTask)
    })

    //! watchers
    watch(
      tasks,
      () => {
        localStorage.setItem('tasks', JSON.stringify(tasks.value))
      },
      { deep: true },
    )

    //! methods
    const addTask = () => {
      if (newTask.value == '' || priority.value == '') {
        errorMessage.value = 'Both the task and priority are required'
        return
      }
      errorMessage.value = null
      tasks.value.unshift({
        id: Math.random(),
        name: newTask.value,
        priority: priority.value as 'High' | 'Medium' | 'Low',
        done: false,
      })
      newTask.value = ''
      priority.value = 'Low'
    }
    const removeTask = (id: number) => {
      tasks.value = tasks.value.filter((task: Task) => task.id != id)
    }

    //! computed
    const filteredTasks = computed<Task[]>(() => {
      if (filter.value == '') {
        return tasks.value
      }

      return tasks.value.filter((task: Task) =>
        task.name.trim().toLowerCase().includes(filter.value.trim().toLowerCase()),
      )
    })
    return {
      newTaskInput,
      tasks,
      priority,
      newTask,
      addTask,
      errorMessage,
      removeTask,
      filteredTasks,
      filter,
    }
  },
}
</script>

<template>
  <div>
    <form @submit.prevent="addTask">
      <!-- Adding tasks form -->
      <input
        type="text"
        ref="newTaskInput"
        placeholder="Add new task"
        v-model="newTask"
        style="margin-right: 5px"
      />
      <select v-model="priority">
        <option disabled value="">Select priority</option>
        <option value="High">High</option>
        <option value="Medium">Medium</option>
        <option value="Low">Low</option>
      </select>
      <span v-if="errorMessage" style="color: red; font-size: 12px; margin-left: 5px">{{
        errorMessage
      }}</span>

      <button type="submit" style="margin: 10px 0px; display: block">Add Task</button>
    </form>

    <!-- Task filtering -->
    <input placeholder="Filter tasks..." v-model="filter" />

    <!-- If there no tasks -->
    <div v-if="tasks.length == 0">
      <p>No tasks found. Try changing the filter or add some tasks!</p>
    </div>

    <div v-else>
      <h3>Your Tasks</h3>
      <!-- This is a list of tasks -->
      <div v-for="task in filteredTasks" :key="task.id">
        <div
          class="task"
          :class="[
            {
              completed: task.done,
            },
            task.priority.toLowerCase(),
          ]"
        >
          <!-- Is task done? -->
          <input type="checkbox" v-model="task.done" />
          <p>{{ task.name }}</p>
          <button class="remove-button" @click="removeTask(task.id)">✖</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
.row {
  display: flex;
  flex-direction: row;
  gap: 5px;
}
body {
  font-family: 'Arial', sans-serif;
  padding: 20px;
  font-size: 16px;
  background-color: #f4f4f9;
}

.task {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: white;
  padding: 10px 20px;
  margin-bottom: 10px;
  border-radius: 5px;
  border-left: 5px solid;
}

.high {
  border-color: red;
}

.medium {
  border-color: orange;
}

.low {
  border-color: green;
}

.completed {
  text-decoration: line-through;
  color: #bbb;
}

.remove-button {
  color: red;
  cursor: pointer;
  border: none;
  background: none;
  font-size: 16px;
}
</style>
