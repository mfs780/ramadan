<template>
  <div id="app">
    <my-header />
    <router-view></router-view>
  </div>
</template>

<script>
import MyHeader from "@/components/MyHeader";

export default {
  name: "app",
  components: {
    MyHeader
  }
};
</script>

<style lang="sass">
html, body, #app
  width: 100%
  height: 100%
  background-color: #F2F2F2 !important;
</style>
