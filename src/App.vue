<template>
  <div id="app">
    <Invest :investList="investList"/>
  </div>
</template>

<script>
import Invest from "./components/Invest.vue";
import Data from "./assets/itemList.json";

export default {
  name: "App",
  components: {
    Invest
  },
  data(){
    return{
      investList: Data
    }
  }
};

</script>

<style lang="scss">
@import url("App.css");
</style>
