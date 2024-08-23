<script setup>
import { onMounted, ref } from 'vue'
import axios from 'axios'

const api = 'https://todolist-api.hexschool.io'
const uid = ref('')
const displayToken = ref('')

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

const token = document.cookie.replace(
  /(?:(?:^|.*;\s*)cookiesSaveToken\s*=\s*([^;]*).*$)|^.*$/,
  '$1'
)

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
  if (signinStatus.value.uid !== '') {
    alert('已登入')
    return
  }
  try {
    const res = await axios.post(`${api}/users/sign_in`, signinData.value)
    displayToken.value = res.data.token
    document.cookie = `cookiesSaveToken = ${res.data.token}`
    location.reload()
  } catch (error) {
    console.log(error.response.data.message)
  }
}

// 驗證
onMounted(async () => {
  try {
    const res = await axios.get(`${api}/users/checkout`, {
      headers: {
        Authorization: token
      }
    })
    signinStatus.value = res.data
  } catch (error) {
    console.log(error.response.data.message)
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
          Authorization: token
        }
      }
    )
    alert('成功登出')
    location.reload()
  } catch (error) {
    alert('登出錯誤: ' + error.message)
  }
}
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
        <p>token: {{ displayToken }}</p>
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
      <h3>登出</h3>
      <div>
        <button type="button" @click="signOut">登出</button>
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
