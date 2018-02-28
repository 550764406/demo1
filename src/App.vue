<template>
  <div id="app">
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>
<style lang="scss">
*{
	padding: 0;
	margin: 0;
  font-family: "Microsoft yahei", "Arial"
}
body,html{
	height: 100%;
    width: 100%;
    overflow: hidden;
}
@font-face {
  font-family: 'sell-icon';
  src:url('common/fonts/sell-icon.eot?exau1o');
  src:url('common/fonts/sell-icon.eot?exau1o#iefix') format('embedded-opentype'),
      url('common/fonts/sell-icon.ttf?exau1o') format('truetype'),
      url('common/fonts/sell-icon.woff?exau1o') format('woff'),
      url('common/fonts/sell-icon.svg?exau1o#sell-icon') format('svg');
  font-weight: normal;
  font-style: normal;
}

[class^="icon-"], [class*=" icon-"] {
  font-family: 'sell-icon' !important;
  speak: none;
  font-style: normal;
  font-weight: normal;
  font-variant: normal;
  text-transform: none;
  line-height: 1;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
