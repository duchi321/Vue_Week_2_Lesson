<script setup>
import { ref } from 'vue'
import axios from 'axios'

const api = 'https://todolist-api.hexschool.io'
const usersignin = ref('')
const userdata = ref({
  email: '',
  password: '',
  nickname: ''
})

const signUp = async () => {
  console.log(`${api}/users/sign_up`)
  try {
    const res = await axios.post(`${api}/users/sign_up`, userdata.value)
    console.log(res)
    usersignin.value = res.data.uid
  } catch (error) {
    console.log(error.response.data.message)
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
        <input type="email" placeholder="email" v-model="userdata.email" /><br />
        <label for="password">密碼: </label>
        <input type="text" placeholder="password" v-model="userdata.password" /><br />
        <label for="nickname">匿稱: </label>
        <input type="text" placeholder="nickname" v-model="userdata.nickname" /><br />
        <button type="button" @click="signUp">註冊</button>
        <p>UID: {{ usersignin }}</p>
        {{ userdata }}
      </div>
    </main>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
}
header {
  /* outline: 3px solid red; */
  width: 100%;
  height: 45px;
  background-color: turquoise;
  box-shadow: 3px 1px 3px 1px rgba(0, 25, 55, 0.6);
}
h1 {
  /* outline: 3px solid red; */
  margin: 0;
  padding: 0 0 0 15px;
  line-height: 45px;
}
</style>
