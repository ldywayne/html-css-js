<template>
    <el-main>
      <router-view :key="key" />
    </el-main>
</template>

<script>
  export default {
    name: 'cmsMain',
    computed: {
      key() {
        return this.$route.path
      }
    }
  }
</script>

<style scoped>
  .el-main {
    min-height: 650px;
  }

  @media screen and (max-width: 768px) {
    .el-main {
      padding: 0px;
    }
  }
</style>
