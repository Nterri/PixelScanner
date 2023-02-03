<template>
<div class="container">
  <h2>ПиксельСканер</h2>
  <scan-picture v-if="showAccept" />
  <div class="error-accept" v-else><span>Приложение не поддерживается на вашем устройстве, откройте повторно при помощи компьютера</span></div>
</div>
</template>

<script>
import ScanPicture from "@/components/ScanPicture";
export default {
  components: {ScanPicture},
  data() {
    return {
      showAccept: true
    }
  },
  mounted() {
    if (/Android|webOS|iPhone|iPad|iPod|BlackBerry|BB|PlayBook|IEMobile|Windows Phone|Kindle|Silk|Opera Mini/i.test(navigator.userAgent)) {
      this.showAccept = false
    } else
      this.showAccept = true
  }
  }
</script>

<style lang="scss" scoped>
@mixin adaptiv-value($property, $startSize, $minSize) {
  $addSize: $startSize - $minSize;
  #{$property}: $startSize + px;
  @media (max-width: #{1024 + px}) {
    #{$property}: calc(#{$minSize + px} + #{$addSize} * ((100vw - 320px) / #{1024 - 320}));
  }
}
h2 {
  text-align: center;
  position: relative;
  z-index: 2;
  font-size: 2vw;
  font-weight: 700;
  @include adaptiv-value('font-size', 24, 16)
  @media (min-width: 1024px) {
    font-size: 2vw;
  }
}
.error-accept {
  flex: 1 1 auto;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 500;
  padding: 0.5vw;
  @include adaptiv-value('font-size', 20, 14)
  text-align: center;
}
</style>