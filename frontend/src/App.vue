<template>
  <div id="app">
    <div id="nav">
      <router-link to="/posts">Posts</router-link> |
      <router-link to="/analyze">Analyze</router-link> |
      <router-link to="/wordclouds">Wordclouds</router-link> |
      <router-link to="/ridgeline">Ridgeline</router-link>
      <!-- <router-link to="/wordcloud">Wordcloud</router-link> -->
    </div>
    <router-view />
  </div>
</template>

<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

#nav {
  padding: 30px;
  text-align: center;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}
</style>
