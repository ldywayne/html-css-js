<template>
  <transition name="fade">
    <div class="input-wrapper" v-if="show">
      <el-input class="gray-bg-input"
      maxlength="100" show-word-limit
                v-model="inputComment"
                type="textarea"
                :rows="3"
                @focus="inputFocus"
                @blur="blur"
                :placeholder="name">
      </el-input>
      <!--enter-active-class="animated fadeInDown" leave-active-class="animated fadeOutUp"-->
      <transition name="fade2">
        <div class="btn-control" v-show="controlShow">
          <el-row>
            <el-col :span="12" style="text-align: left">
              <Emoji @output="output"></Emoji>
            </el-col>
            <el-col :span="12" style="text-align: right">
              <span class="cancel" @click="cancel">取消</span>
              <el-button class="btn" type="success" round @click="commitComment">确定</el-button>
            </el-col>
          </el-row>
        </div>
      </transition>
    </div>
  </transition>
</template>

<script>
  import Emoji from '@/components/Emoji'
  export default {
    props: {
      //控制整个组件是否显示
      show: {
        type: Boolean,
        required: true,
      },
      //传入input框的默认值
      value: {
        type: String
      },
      //传入input框的默认值
      toComment: {
        type: String
      },
      //传入input框的默认值
      toId: {
        type: Number
      },
      //类型，end(文章末尾处), comment(评论里),
      type: {
        type: String,
        // default: 'comment'
      }
    },
    components: {
      Emoji
    },
    data() {
      return {
        inputComment: '',
        name:'',
        id:'',
        //确定取消按钮是否显示
        controlShow: false,
        cursorIndexStart: null,//光标选中开始的位置
        cursorIndexEnd: null,//光标选中结束的位置
      }
    },
    computed: {},
    methods: {
      /**
       * 点击取消按钮
       */
      cancel() {
        if (this.type === 'end') {
          this.controlShow = false
        }
        this.$emit("cancel")
      },

      /**
       * 提交评论
       */
      commitComment() {
        this.$emit("confirm", {'inputComment':this.inputComment,'id':this.id})
        this.inputComment = ""
      },

      //input活得焦点时调用
      inputFocus() {
        // console.log("focus");
        if (this.type === 'end') {
          this.controlShow = true
        }
      },
      blur(e){
        this.cursorIndexStart = e.srcElement.selectionStart  // 获取input输入框失去焦点时光标选中开始的位置
        this.cursorIndexEnd = e.srcElement.selectionEnd  // 获取input输入框失去焦点时光标选中结束的位置
      },
      output(val) {
        if (this.cursorIndexStart !== null && this.inputComment) {
          //如果 文本域获取了焦点, 则在光标位置处插入对应字段内容
          this.inputComment = this.inputComment.substring(0, this.cursorIndexStart) + val + this.inputComment.substring(this.cursorIndexEnd)
        } else {
          // 如果 文本域未获取焦点, 则在字符串末尾处插入对应字段内容
          this.inputComment = this.inputComment?this.inputComment:'' + val
        }
      },

    },
    watch: {
      //监听toComment更新，赋值给name
      toComment: function (newValue, oldValue) {
        this.name = newValue;
        this.inputComment = ''
      },
      toId: function (newValue, oldValue) {
        this.id = newValue;
        this.inputComment = ''
      }
    },
    mounted() {
      if (this.type === 'end') {
        this.controlShow = false
      } else {
        this.controlShow = true
      }
      // console.log(this.controlShow)
    }
  }
</script>

<style scoped rel="stylesheet/scss" lang="scss">

  .fade-enter-active, fade-leave-active {
    transition: opacity 0.5s;
  }

  .fade-enter, .fade-leave-to {
    opacity: 0;
  }

  .input-wrapper {
    padding: 10px;

    .fade2-enter-active, fade2-leave-active {
      transition: opacity 0.5s;
    }

    .fade2-enter, .fade2-leave-to {
      opacity: 0;
    }
    .gray-bg-input, .el-input__inner {
      /*background-color: #67C23A;*/
    }
    .btn-control {
      justify-content: flex-end;
      align-items: center;
      padding-top: 10px;
      .cancel {
        font-size: 16px;
        color: #606266;
        margin-right: 20px;
        cursor: pointer;
        &:hover {
          color: #333;
        }
      }
      .confirm {
        font-size: 16px;
      }
    }
  }
</style>
