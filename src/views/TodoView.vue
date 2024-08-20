<template>
  <div>
    <h1>This is an Todo page</h1>
  </div>

  <div>
    <h2>註冊</h2>
    <input type="email" placeholder="Email" v-model="signupField.email" />
    <input type="text" placeholder="Passwoerd" v-model="signupField.password" />
    <input type="text" placeholder="nickname" v-model="signupField.nickname" />
    {{ signupField }}
    <button type="button" @click="signup">註冊</button>
    UID: {{ signupRes }}
  </div>

  <div>
    <h2>登入</h2>
    <input type="email" placeholder="Email" v-model="signInField.email" />
    <input type="text" placeholder="Passwoerd" v-model="signInField.password" />
    {{ signInField }}
    <button type="button" @click="signIn">登入</button>
    UID: {{ signInRes }}
  </div>

  <div>
    <h2>驗證</h2>
    <div v-if="user.uid">
      <p>UID: {{ user.uid }}</p>
      <p>nickname: {{ user.nickname }}</p>
    </div>
    <div v-else>你還沒有登入</div>
  </div>

  <div>
    <h2>登出</h2>
    <input type="text" v-model="tokenSignOut" placeholder="Token" />
    <button type="button" @click="signOut">登出</button>
  </div>

  <!-- Todo list -->
  <h2>Todo list</h2>
  <div v-if="token">
    <input type="text" placeholder="New Todo" v-model="newTodo" />
    <button type="button" @click="addTodo">Add Todo</button>
    <ul>
      <li v-for="(todo, index) in todos" :key="index">
        {{ todo.content }} {{ todo.status ? '完成' : '未完成' }} | {{ todoEdit[todo.id] }}
        <input type="text" placeholder="更新值" @change="updateTodoEdit($event, todo.id)" />
        <button @click="deleteTodo(todo.id)">Delete</button>
        <button @click="updateTodo(todo.id)">Update</button>
        <button @click="toggleStatus(todo.id)">Toggle Status</button>
      </li>
    </ul>
  </div>
</template>

<script setup>
import axios from 'axios'
import { onMounted, ref } from 'vue'

const api = 'https://todolist-api.hexschool.io'

// 註冊
const signupField = ref({
  email: '',
  password: '',
  nickname: ''
})

const signupRes = ref('')

const signup = async () => {
  console.log(`${api}/users/sign_up`)

  try {
    const res = await axios.post(`${api}/users/sign_up`, signupField.value)

    console.log(res)
    signupRes.value = res.data.uid
  } catch (error) {
    console.log(error)
  }
}

// 登入
const signInField = ref({
  email: '',
  password: ''
})

const signInRes = ref('')

const signIn = async () => {
  try {
    const res = await axios.post(`${api}/users/sign_in`, signInField.value)

    console.log(res)
    signInRes.value = res.data.token

    document.cookie = `customTodoToken=${res.data.token}`
  } catch (error) {
    console.log(error)
  }
}

// 驗證登入
const user = ref({
  nickname: '',
  uid: ''
})

onMounted(async () => {
  const token = document.cookie.replace(
    /(?:(?:^|.*;\s*)customTodoToken\s*=\s*([^;]*).*$)|^.*$/,
    '$1'
  )
  console.log(token)

  const res = await axios.get(`${api}/users/checkout`, {
    headers: {
      Authorization: token
    }
  })
  // console.log(res)
  user.value = res.data
})

// 登出
const tokenSignOut = ref('')

const signOut = async () => {
  try {
    const res = await axios.post(
      `${api}/users/sign_out`,
      {},
      {
        headers: {
          Authorization: tokenSignOut.value
        }
      }
    )

    console.log(res.data)
  } catch (error) {
    console.log(error)
  }
}

// Todo list
const todos = ref([])
const newTodo = ref('')
const todoEdit = ref({})
const token = ref('')

// 在組件掛載時獲取 token
onMounted(() => {
  token.value = document.cookie.replace(
    /(?:(?:^|.*;\s*)customTodoToken\s*=\s*([^;]*).*$)|^.*$/,
    '$1'
  )
  if (token.value) {
    getTodos()
  }
})

const getTodos = async () => {
  const res = await axios.get(`${api}/todos`, {
    headers: {
      Authorization: token.value
    }
  })
  todos.value = res.data.data
}

const addTodo = async () => {
  if (!newTodo.value) return

  const todo = {
    content: newTodo.value
  }
  await axios.post(`${api}/todos`, todo, {
    headers: {
      Authorization: token.value
    }
  })
  newTodo.value = ''
  getTodos()
}

const deleteTodo = async (id) => {
  await axios.delete(`${api}/todos/${id}`, {
    headers: {
      Authorization: token.value
    }
  })
  getTodos()
}

const updateTodo = async (id) => {
  const todo = todos.value.find((todo) => todo.id === id)
  todo.content = todoEdit.value[id]
  await axios.put(`${api}/todos/${id}`, todo, {
    headers: {
      Authorization: token.value
    }
  })
  getTodos()
  todoEdit.value = {
    ...todoEdit.value,
    [id]: ''
  }
}

const toggleStatus = async (id) => {
  await axios.patch(
    `${api}/todos/${id}/toggle`,
    {},
    {
      headers: {
        Authorization: token.value
      }
    }
  )
  getTodos()
}

const updateTodoEdit = (event, id) => {
  todoEdit.value = {
    ...todoEdit.value,
    [id]: event.target.value
  }
}

const TodoToken = document.cookie
  .split('; ')
  .find((row) => row.startsWith('hexschoolTodo='))
  ?.split('=')[1]

onMounted(() => {
  if (TodoToken) {
    token.value = TodoToken
  }
})
</script>
