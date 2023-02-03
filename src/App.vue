<template>
    <div class="content">
      <router-view>

      </router-view>
    </div>
</template>

<script>
export default {
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: inherit;
}
@import url(https://fonts.googleapis.com/css?family=Montserrat:300,regular,500,600,italic&display=swap);
#app {
  font-style: normal;
  font-family: Montserrat;
  color: black;
  background: #fff;
  display: flex;
  min-height: 100vh;
  flex-direction: column;
}
.content {
  flex: 1 1 auto;
  display: flex;
  align-items: center;
  justify-content: center;
}
.container {
  min-height: calc(100vh - 30px);
  margin: 0 auto;
  padding: 1vw;
  box-sizing: content-box;
  color: black;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
</style>