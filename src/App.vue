<template>
  <img alt="Vue logo" src="./assets/cagale.png">
  <LoginComponent 
    v-if=!hasToken
    @refreshData=getToken()
  />
  <PageComponent
    v-else
  />
</template>

<script>
import LoginComponent from './components/LoginComponent.vue'
import PageComponent from './components/PageComponent.vue'

export default {
  name: 'App',
  components: {
    LoginComponent,
    PageComponent
  },
  data() {
    return {
     hasToken:false
    };
  },
  mounted() {
    console.log(localStorage.getItem('authToken'))
    this.hasToken = localStorage.getItem('authToken') !== null && localStorage.getItem('authToken') !== '';
  },
  methods: {
    getToken () {
      console.log("getToken")
      this.hasToken = localStorage.getItem('authToken') !== null && localStorage.getItem('authToken') !== ''
    }
  }

  
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
