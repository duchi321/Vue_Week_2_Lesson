<script setup>
import { onMounted, ref } from 'vue'
import axios from 'axios'

const api = 'https://todolist-api.hexschool.io'
const uid = ref('')
const showToken = ref('')
const tokenCheck = ref('')
const messageSignUp = ref('')
const userData = ref({
  email: '',
  password: '',
  nickname: ''
})

const signinData = ref({
  email: '',
  password: ''
})

const signinStatus = ref({
  nickname: '',
  uid: ''
})

const todes = ref([])
const newTodo = ref('')
const tempTodoThing = ref({
  id: '',
  content: ''
})

// 註冊
const signUp = async () => {
  try {
    const res = await axios.post(`${api}/users/sign_up`, userData.value)
    uid.value = res.data.uid
    messageSignUp.value = `註冊成功! UID: ${res.data.uid}`
    alert(messageSignUp.value)
  } catch (error) {
    messageSignUp.value = `註冊失敗! 格式錯誤!?`
    alert(messageSignUp.value)
  }
}

// 登入
const signIn = async () => {
  if (signinStatus.value.uid !== '') {
    alert('已登入')
    return
  }
  try {
    const res = await axios.post(`${api}/users/sign_in`, signinData.value)
    showToken.value = res.data.token
    document.cookie = `cookiesSaveToken = ${res.data.token}`
    location.reload()
    alert('登入')
  } catch (error) {
    alert('帳號或密碼錯誤!')
  }
}

// 驗證
onMounted(async () => {
  const token = document.cookie.replace(
    /(?:(?:^|.*;\s*)cookiesSaveToken\s*=\s*([^;]*).*$)|^.*$/,
    '$1'
  )
  tokenCheck.value = token
  document.cookie = `cookiesSaveToken = ${tokenCheck.value}`
  try {
    const res = await axios.get(`${api}/users/checkout`, {
      headers: {
        Authorization: tokenCheck.value
      }
    })
    signinStatus.value = res.data
  } catch (error) {
    console.log('未登入')
  }
})

// 登出
const signOut = async () => {
  if (signinStatus.value.uid === '') {
    alert('未登入')
    return
  }
  try {
    await axios.post(
      `${api}/users/sign_out`,
      {},
      {
        headers: {
          Authorization: tokenCheck.value
        }
      }
    )
    alert('成功登出')
    location.reload()
  } catch (error) {
    alert('登出錯誤: ' + error.message)
  }
}

// Todo List
const getTodos = async () => {
  const res = await axios.get(`${api}/todos/`, {
    headers: {
      Authorization: tokenCheck.value
    }
  })
  todes.value = res.data.data
}

const createTodos = async () => {
  if (newTodo.value) {
    await axios.post(
      `${api}/todos/`,
      {
        content: newTodo.value
      },
      {
        headers: {
          Authorization: tokenCheck.value
        }
      }
    )
    newTodo.value = ''
    getTodos()
  } else {
    // 沒有填寫代辦事項
  }
}

const deleteTodo = async (id) => {
  await axios.delete(`${api}/todos/${id}`, {
    headers: {
      Authorization: tokenCheck.value
    }
  })
  getTodos()
}

const editTodos = async (id) => {
  const index = todes.value.findIndex((item) => item.id === id)
  tempTodoThing.value = {
    id: id,
    content: todes.value[index].content
  }
}

const updateTodos = async (id) => {
  if (tempTodoThing.value.content !== '') {
    await axios.put(
      `${api}/todos/${id}`,
      {
        content: tempTodoThing.value.content
      },
      {
        headers: {
          Authorization: tokenCheck.value
        }
      }
    )
    tempTodoThing.value = { id: '', content: '' }
    getTodos()
  } else {
    alert('請填寫項目')
  }
}

const toggleTodo = async (id) => {
  await axios.patch(
    `${api}/todos/${id}/toggle`,
    {},
    {
      headers: {
        Authorization: tokenCheck.value
      }
    }
  )
  getTodos()
}

const TodoToken = document.cookie
  .split('; ')
  .find((row) => row.startsWith('cookiesSaveToken='))
  ?.split('=')[1]

onMounted(() => {
  if (tokenCheck.value) {
    getTodos()
  }

  if (TodoToken) {
    tokenCheck.value = TodoToken
  }
})
</script>

<template>
  <div class="container">
    <header>
      <h1>week two homework</h1>
    </header>
    <main>
      <div>
        <h2>註冊</h2>
        <label for="email">信箱: </label>
        <input type="email" placeholder="email" v-model="userData.email" /><br />
        <label for="password">密碼: </label>
        <input type="text" placeholder="password" v-model="userData.password" /><br />
        <label for="nickname">匿稱: </label>
        <input type="text" placeholder="nickname" v-model="userData.nickname" /><br />
        <button type="button" @click="signUp">註冊</button>
        <p>UID: {{ uid }}</p>
        {{ userData }}
      </div>
      <hr />
      <div>
        <h2>登入</h2>
        <label for="email">信箱: </label>
        <input type="email" placeholder="email" v-model="signinData.email" /><br />
        <label for="password">密碼: </label>
        <input type="text" placeholder="password" v-model="signinData.password" /><br />
        <button type="button" @click="signIn">登入</button>
        <p>token: {{ showToken }}</p>
        {{ signinData }}
      </div>
      <hr />
      <div>
        <h2>驗證</h2>
        <div v-if="signinStatus.uid">
          <p>暱稱: {{ signinStatus.nickname }}</p>
          <p>UID: {{ signinStatus.uid }}</p>
        </div>
        <div v-else>目前未登入</div>
      </div>
      <hr />
      <h2>登出</h2>
      <div>
        <button type="button" @click="signOut">登出</button>
      </div>
      <hr />
      <h2>Todo list</h2>
      <div v-if="signinStatus.nickname">
        <div class="todolistgroup">
          <div>
            <input type="text" placeholder="待辦事項  " v-model="newTodo" />
            <button type="button" @click="createTodos">新增</button>
          </div>
          <ul>
            <li v-for="todo in todes" :key="todo.id">
              <div>
                <input type="checkbox" v-model="todo.status" @change="toggleTodo(todo.id)" />
                <span
                  v-show="tempTodoThing.id !== todo.id"
                  :class="{ 'line-through': todo.status }"
                  >{{ todo.content }}</span
                >
                <input
                  type="text"
                  v-if="tempTodoThing.id === todo.id"
                  v-model="tempTodoThing.content"
                />
                <button
                  type="button"
                  v-if="tempTodoThing.id === todo.id"
                  @click="updateTodos(todo.id)"
                >
                  確定
                </button>
              </div>
              <div>
                <button type="button" @click="editTodos(todo.id)">修改</button>
                <button type="button" @click="deleteTodo(todo.id)">刪除</button>
              </div>
            </li>
          </ul>
        </div>
      </div>
      <div v-else>請登入帳號</div>
    </main>
  </div>
</template>

<style scoped>
* {
  margin: 0;
  padding: 0;
}

.container {
  /* outline: 3px solid red; */
  display: flex;
  flex-direction: column;
}

header {
  /* outline: 3px solid red; */
  height: 45px;
  background-color: turquoise;
  box-shadow: 3px 1px 3px 1px rgba(0, 25, 55, 0.6);
}

h1 {
  padding: 0 0 0 15px;
  line-height: 45px;
}

/* todoList */
.todolistgroup {
  /* outline: 3px solid red; */
  width: 400px;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  border-bottom: 1px solid #ccc;
}

input {
  margin-right: 10px;
}

button {
  margin-right: 5px;
  padding: 3px 8px;
  background-color: #72777b;
  color: white;
  border: none;
  border-radius: 3px;
  cursor: pointer;
}

button:hover {
  background-color: #218838;
}

button:nth-child(2):hover {
  background-color: #c82333;
}

.line-through {
  text-decoration: line-through;
  color: #6c757d;
}
</style>
