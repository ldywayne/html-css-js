<template>
  <el-popover placement="bottom" trigger="click" ref="popover"
    v-model="popoverVisible">
    <div class="emjioBox">
      <ul class="emjio">
        <li class="emjioLi" v-for="(item,i) in this.emojiList" :key="i" @click="selectEmit(item)">
          {{item}}
        </li>
      </ul>
    </div>
    <el-link :underline="false" title="添加表情"
      slot="reference">
      😀
    </el-link>
  </el-popover>
</template>

<script>
import emojiList from './emoji.json';
export default {
  name: 'emoji',
  props: {},
  data() {
    return {
      emojiList: [],
      popoverVisible: false,
    }
  },
  watch: {},
  created() {
    this.init();
  },
  methods: {
    init() {
      this.emojiList = emojiList.data
    },
    // 选择表情
    selectEmit(item) {
      this.$emit('output', item)
      this.popoverVisible = false
    }
  },
  beforeDestroy() {

  }
}
</script>

<style lang="scss" scoped>
  .emjioBox {
    background: #fff;
    height: 150px;
    width: 300px;
    overflow: auto;
    text-align: left;
  }
  .emjioBox .emjio {
    padding: 0;
  }
  .emjioBox li {
    display: inline-block;
    width: 28px;
    height: 28px;
    line-height: 28px;
    text-align: center;
    cursor: pointer;
  }
</style>
