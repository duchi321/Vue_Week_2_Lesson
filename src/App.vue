<script setup>
import { onMounted, ref } from 'vue'
import axios from 'axios'

const api = 'https://todolist-api.hexschool.io'
const uid = ref('')
const userToken = ref('')

const userData = ref({
  email: '',
  password: '',
  nickname: ''
})
const signinData = ref({
  email: '',
  password: ''
})

const signinStatusData = ref({
  nickname: '',
  uid: ''
})

// 註冊
const signUp = async () => {
  try {
    const res = await axios.post(`${api}/users/sign_up`, userData.value)
    console.log(res)
    uid.value = res.data.uid
  } catch (error) {
    console.log(error.response.data.message)
  }
}

// 登入
const signIn = async () => {
  try {
    const res = await axios.post(`${api}/users/sign_in`, signinData.value)
    console.log(res.data)
    userToken.value = res.data.token
    document.cookie = `cookiesSaveToken = ${res.data.token}`
  } catch (error) {
    console.log(error.response.data.message)
  }
}

// 驗證
onMounted(async () => {
  const token = document.cookie.replace(
    /(?:(?:^|.*;\s*)cookiesSaveToken\s*=\s*([^;]*).*$)|^.*$/,
    '$1'
  )
  console.log('token: ', token)
  const res = await axios.get(`${api}/users/checkout`, {
    headers: {
      Authorization: token
    }
  })
  console.log(res)
  signinStatusData.value = res.data
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
        <p>token: {{ userToken }}</p>
        {{ signinData }}
      </div>
      <hr />
      <div>
        <h2>驗證</h2>
        <div v-if="signinStatusData.uid">
          <p>暱稱: {{ signinStatusData.nickname }}</p>
          <p>UID: {{ signinStatusData.uid }}</p>
        </div>
        <div v-else>目前未登入</div>
      </div>
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
</style>
