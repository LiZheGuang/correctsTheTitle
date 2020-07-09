<template>
  <div id="app">
    <correcting
      ref="correct"
      :commitImageUrls="list"
      isPageView="edit"
    ></correcting>
  </div>
</template>

<script>
import correcting from "./components/correcting.vue";

export default {
  name: "App",
  components: {
    correcting
  },
  data() {
    return {
      list: []
    };
    
  },
  mounted() {
    this.list = [require('./assets/demo/h5_optimize_share_topicSelect_choose_item_1_normal@3x.png')]
  }
};
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
