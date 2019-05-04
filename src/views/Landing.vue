<template>
  <div id="landing">
    <my-header/>
    <router-view></router-view>
  </div>
</template>

<script>
import MyHeader from "@/components/MyHeader";

export default {
  name: "Landing",
  components: {
    MyHeader
  }
};
</script>

<style scoped lang="sass">
#landing
  width: 100%
  height: 100%
</style>
