<template>
  <el-container style="opacity: 0.9">
    <div class="author">
      <el-avatar v-if="token==null" icon="el-icon-user-solid" size="large">
        <!-- style="background-color: #666" -->
      </el-avatar>
      <el-avatar v-else :src="avatar" size="large"></el-avatar>
      <div>
        <div class="nkname">
          <span class="name" v-if="token==null">匿名用户</span>
          <span class="name" v-else>{{name}} </span>
        </div>
      </div>
    </div>
    <el-form :model="messageForm" :rules="messageFormRules" ref="messageFormRef">
      <el-form-item prop="content">
        <el-input @blur="blur" :rows="5" v-model="messageForm.content" type="textarea" maxlength="100" show-word-limit
          placeholder="请输入你的评论"></el-input>
      </el-form-item>
      <el-form-item style="text-align: right">
        <el-row>
          <el-col :span="12" style="text-align: left">
            <Emoji @output="output"></Emoji>
          </el-col>
          <el-col :span="12" style="text-align: right">
            <el-button type="primary"  v-hasPermi="['cms:comment:add']" @click="publish">点击发表</el-button>
          </el-col>
        </el-row>
      </el-form-item>
    </el-form>
    <el-divider v-if="messageList.length>0"><span style="color: #999;font-size: small;">最新评论</span></el-divider>
    <comment :comments="messageList" @replyConfirm="commitComment"></comment>
    <pagination v-show="total>0" :total="total" :page.sync="queryParams.pageNum" :limit.sync="queryParams.pageSize"
      @pagination="getMessageList" />
  </el-container>
</template>

<script>
  import {
    mapGetters
  } from 'vuex'
  import {
    getToken
  } from '@/utils/auth'
  import {
    cmsListComment,
    cmsAddComment,
  } from "@/api/cms/comment"
  import comment from './comments.vue'
  import Emoji from '@/components/Emoji'
  export default {
    name: 'Ipcomment',
    data() {
      return {
        picList: [],
        editing: false,
        messageList: [],
        // userInfo: null,
        message: {
          userId: -1,
          content: ''
        },
        messageForm: {},
        // 总条数
        total: 0,
        // 查询参数
        queryParams: {
          pageNum: 1,
          pageSize: 10,
          parentId: null,
          mainId: null,
          likeNum: null,
          content: null,
          type: null,
          blogId: this.$route.query.id,
          userId: null,
          delFlag: null,
          createBy: null,
        },
        messageFormRules: {
          content: [{
            min: 0,
            max: 100,
            message: "评论内容不超过100字！"
          }]
        },
        cursorIndexStart: null,//光标选中开始的位置
        cursorIndexEnd: null,//光标选中结束的位置
      }
    },
    created() {
      this.getMessageList()
      this.reset();
    },
    updated: function() {
      this.$nextTick(function() {
        // 仅在整个视图都被渲染之后才会运行的代码
        this.to();
      })
    },
    computed: {
      ...mapGetters([
        'token',
        'avatar',
        'name'
      ]),
    },
    components: {
      comment,
      Emoji
    },
    methods: {
      // 表单重置
      reset() {
        this.messageForm = {
          id: null,
          parentId: null,
          mainId: null,
          likeNum: null,
          content: null,
          type: null,
          blogId: this.$route.query.id,
          userId: null,
          delFlag: null,
          createBy: null,
          createTime: null,
          updateBy: null,
          updateTime: null
        };
        this.resetForm("messageForm");
      },
      // 评论发表
      publish() {
        let token = getToken();
        this.$refs.messageFormRef.validate(async valid => {
          if (!valid) return
          if (this.messageForm.content == null || this.messageForm.content == '') {
            this.$modal.msgError("评论内容不能为空！");
            return;
          }
          if (token == null || token == '') {
            this.messageForm.createBy = "匿名用户"
            this.messageForm.type = '0'
          } else {
            this.messageForm.createBy = this.$store.getters.name
            this.messageForm.type = '0'
          }
          cmsAddComment(this.messageForm).then(response => {
            this.$modal.msgSuccess("评论发表成功");
            this.reset();
            this.getMessageList();
          });
        })
      },
      /**
       * 提交评论
       */
      commitComment(value) {
        this.reset();
        this.messageForm.content = value.inputComment;
        this.messageForm.parentId = value.id;
        let token = getToken();
        this.$refs.messageFormRef.validate(async valid => {
          if (!valid) return
          if (this.messageForm.content == null || this.messageForm.content == '') {
            this.$modal.msgError("评论内容不能为空！");
            return;
          }
          if (token == null || token == '') {
            this.messageForm.createBy = "匿名用户"
            this.messageForm.type = '1'
          } else {
            this.messageForm.createBy = this.$store.getters.name
            this.messageForm.type = '1'
          }
          cmsAddComment(this.messageForm).then(response => {
            this.$modal.msgSuccess("评论发表成功");
            this.reset();
            this.getMessageList();
          });
        })
      },
      // 获取评论列表
      async getMessageList() {
        let token = getToken();
        if (token != null && token != '') {
          this.queryParams.createBy = this.$store.getters.name
        }
        cmsListComment(this.queryParams).then(response => {
          for (let i = 0; i < response.rows.length; i++) {
            let mesInfo = response.rows[i];
            if (mesInfo.avatar != null && mesInfo.avatar != "") {
              response.rows[i].avatar = process.env.VUE_APP_BASE_API + mesInfo.avatar
            }
            if (mesInfo.children != null && mesInfo.children != "") {
              for (let j = 0; j < response.rows[i].children.length; j++) {
                let children = response.rows[i].children;
                if (children.avatar != null && children.avatar != "") {
                  response.rows[i].children[j].avatar = process.env.VUE_APP_BASE_API + children.avatar
                }
              };
            }
          };
          this.messageList = response.rows;
          this.total = response.total;
        });
      },
      blur(e){
        this.cursorIndexStart = e.srcElement.selectionStart  // 获取input输入框失去焦点时光标选中开始的位置
        this.cursorIndexEnd = e.srcElement.selectionEnd  // 获取input输入框失去焦点时光标选中结束的位置
      },
      output(val) {
        if (this.cursorIndexStart !== null && this.messageForm.content) {
          //如果 文本域获取了焦点, 则在光标位置处插入对应字段内容
          this.messageForm.content = this.messageForm.content.substring(0, this.cursorIndexStart) + val + this.messageForm.content.substring(this.cursorIndexEnd)
        } else {
          // 如果 文本域未获取焦点, 则在字符串末尾处插入对应字段内容
          this.messageForm.content = this.messageForm.content?this.messageForm.content:'' + val
        }
      },
      //跳转到相应位置
      to() {
        if (this.$route.query.commentId != null) {
          var toEl = document.getElementById(this.$route.query.commentId);
          if (toEl != null) {
            if (toEl != null && toEl != "") {
              // toEl 为指定跳转到该位置的DOM节点
              let bridgeCms = toEl;
              let bodyTop = document.body;
              let heightCms = 0;
              // 计算该 DOM 节点到 bodyTop 顶部距离
              do {
                heightCms += bridgeCms.offsetTop;
                bridgeCms = bridgeCms.offsetParent;
              } while (bridgeCms !== bodyTop)
              // 滚动到指定位置
              window.scrollTo({
                top: heightCms,
                behavior: 'smooth'
              })
            }
          }
        }
      },

    },
  }
</script>

<style scoped>
  .el-container {
    display: block;
  }

  .author {
    display: flex;
    justify-content: flex-start;
    align-items: center;
    width: 100%;
    margin-bottom: 20px;
  }

  .comment {
    border-bottom: 1px dashed #ccc;
    margin: 30px 0;
    display: flex;
  }

  .content {
    text-align: left;
    font-size: 14px;
    flex-grow: 1;
  }

  .nkname {
    margin: 10px;
    max-width: 530px;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
  }

  .date {
    color: #999;
    margin-left: 10px;
  }

  .reply {
    margin-left: 10px;
  }
</style>
