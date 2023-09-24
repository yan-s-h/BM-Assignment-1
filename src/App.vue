<template>
  <div class="manual">
    <h2>Instructions:</h2>
    <p>Welcome to the To-Do List! Here's how to use it:</p>
    <ol>
      <li>Enter a task in the input field.</li>
      <li>Click the "Add" button to add the task to your to-do list.</li>
      <li>To mark a task as done, click the checkbox next to it.</li>
      <li>To edit a task, click the pencil icon, make your changes, and press Enter to save. You can also press the Esc
        key to cancel the edit.</li>
      <li>To delete a task, click the trash can icon.</li>
    </ol>
  </div>
  <form @submit.prevent="addItem">
    <input type="text" v-model="newItem">
    <input type="submit" value="Add">
  </form>
  <br>
  <table style="border: 1px;">
    <tr>
      <td>
        <h1>To Do</h1>

      </td>
      <td>
        <h1>Done</h1>
      </td>
    </tr>
    <tr>
      <td>
        <ul>
          <li v-for="item in items" :key="item.id">
            <input type="checkbox" @click="itemDone(item.id)" v-model="item.done" :checked="item.checked">
            <span v-if="item.edit">
              <input type="text" v-model="item.model" @keydown.enter="saveEditItem(item.id)"
                @keydown.esc="cancelEditItem(item.id)">
            </span>
            <span v-else>
              {{ item.name }}
              <span @click="editItem(item.id)"><i class="fas fa-edit"></i></span>
            </span>
            <span @click="deleteItem(item.id)"><i class="fas fa-trash"></i></span>
          </li>
        </ul>
      </td>
      <td>
        <ul>
          <li v-for="item in itemsDone" :key="item.id">
            <input type="checkbox" @click="itemToDo(item.id)" v-model="item.done" :checked="item.checked">{{ item.name }}
          </li>
        </ul>

      </td>
    </tr>

  </table>
</template>

<script setup>
import { ref, reactive, watch, onMounted } from 'vue'
import '@fortawesome/fontawesome-free/css/all.css'
import '@fortawesome/fontawesome-free/js/all.js'

const newItem = ref('')
const items = reactive([])
const id = ref(1)
const itemsDone = reactive([])

const addItem = () => {
  if (newItem.value.length < 2) return

  items.push({
    id: id.value++,
    name: newItem.value,
    done: false,
    edit: false,
    model: newItem.value

  })

  newItem.value = ''
}

const findIndexById = id => {
  return items.findIndex(item => item.id === id)
}
const findIndexByIdDone = id => {
  return itemsDone.findIndex(item => item.id === id)
}

const deleteItem = id => {
  const idx = findIndexById(id)
  if (idx > -1) {
    items.splice(idx, 1)
  }
}

const editItem = id => {
  const idx = findIndexById(id)
  if (idx > -1) {
    items[idx].edit = true
  }
}
const saveEditItem = id => {
  const idx = findIndexById(id)
  if (idx > -1) {
    items[idx].name = items[idx].model
    items[idx].edit = false
  }
}
const cancelEditItem = id => {
  const idx = findIndexById(id)
  if (idx > -1) {
    items[idx].model = items[idx].name
    items[idx].edit = false
  }
}
const itemDone = id => {
  const idx = findIndexById(id)

  if (idx > -1) {
    const itemToMove = items[idx]
    items.splice(idx, 1)
    itemToMove.done = true
    itemToMove.checked = true
    itemsDone.push(itemToMove)
  }
}
const itemToDo = id => {
  const idxDone = findIndexByIdDone(id)
  if (idxDone > -1) {
    const itemToMoveBack = itemsDone[idxDone]
    itemsDone.splice(idxDone, 1)
    itemToMoveBack.done = false
    itemToMoveBack.checked = false
    items.push(itemToMoveBack)
  }
}
watch([items, itemsDone], () => {
  localStorage.vueNewTodo = JSON.stringify(items)
  localStorage.vueNewDone = JSON.stringify(itemsDone)
})

onMounted(() => {
  Object.assign(items, JSON.parse(localStorage.vueNewTodo || '[]'))
  Object.assign(itemsDone, JSON.parse(localStorage.vueNewDone || '[]'))

  if (items.length > 0 || itemsDone.length > 0) {
    const maxId = Math.max(
      ...items.map(item => item.id),
      ...itemsDone.map(item => item.id)
    )
    id.value = maxId + 1
    console.log(maxId)
  }
})
</script>

<style scoped lang="sass">
*
  font-family: Arial, Helvetica, sans-serif
h1
  text-align: center
table
  border-collapse: collapse
  margin: auto

td
  border: 1px solid black
  width: 250px
form
  margin: auto
  width: 500px
.manual
  margin: auto
  width: 500px
</style>
