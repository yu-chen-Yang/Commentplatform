<template>
  <header class="editor-header">
    <router-link class="m-r-20 logo-wrapper" :to="{ name: 'home' }">
      <img class="logo" src="./../assets/imgs/logo.png" alt="" />
    </router-link>
    <div class="header-content">
      <span class="title">写文章</span>
    </div>
    <div class="functions">
      <el-button
        type="primary"
        plain
        size="large"
        @click="$emit('release-articles')"
        class="publish m-r-25"
      >
        发布
      </el-button>
    </div>
  </header>
</template>

<script>
//import request from "@/service";

export default {
  data() {
    return {
      name: "",
      id: 0,
      isLogin: false
    };
  },
  mounted() {
    // this.checkLogin();
  },
  methods: {
    checkLogin: function() {
      this.$axios.get("/users/checkLogin").then(res => {
        if (res.data.status === 200) {
          this.name = res.data.data.userName;
          this.id = res.data.data.userID;
          this.isLogin = true;
        } else {
          this.$router.push({ name: "signup" });
          this.isLogin = false;
        }
      });
    }
  }
};
</script>

<style></style>
