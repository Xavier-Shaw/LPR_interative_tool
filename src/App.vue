<template>
  <privacy-review></privacy-review>
</template>

<script>
import PrivacyReview from "@/components/privacyReview";

export default {
  name: 'App',
  components: {
    PrivacyReview
  }
}
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
