<template>
  <div id="app">
    <tcs :mapSize="mapSize" :boxSize="boxSize"/>
  </div>
</template>
<script>
import tcs from './views/Tcs'
export default {
  data(){
    return {
      mapSize:40,//个数 
      boxSize:20//px 单个小方块大小
    }
  },
  components:{
    tcs
  }
}
</script>
<style lang="less">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#nav {
  padding: 30px;

  a {
    font-weight: bold;
    color: #2c3e50;

    &.router-link-exact-active {
      color: #42b983;
    }
  }
}
</style>
