<template>
  <div>
    <Header />

    <div style="display: flex">

      <Aside/>

      <router-view style="flex: 1" @userInfo="refreshUser"/>
    </div>
  </div>
</template>

<script>
import Header from "../components/Header";
import Aside from "../components/Aside";
import request from "../utils/request";


export default {
  name: "Layout",
  components:{
    Header,
    Aside
  },
  data() {
    return {
      user: {}
    }
  },
  methods: {
    refreshUser() {
      let userJson = sessionStorage.getItem("user");
      if (!userJson) {
        return
      }
      let userId = JSON.parse(userJson).id
      request.get("/user/" + userId).then(res => {
        this.user = res.data
      })
    }
  }
}
</script>

<style scoped>

</style>