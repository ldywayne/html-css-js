<template>
  <el-row :gutter="20">
    <el-col :sm="2" class="hidden-xs-only" style="opacity:0;">左侧占位</el-col>
    <el-col :xs="24" :sm="15">
      <el-card style="background-color: rgba(255,255,255,0.9)" class="left-item">
        <div slot="header" class="total">
          <div class="titleIndex">
            <i v-if="selected" class="el-icon-back" @click="updateBlogList"></i>
            <span>{{selectMethod}}</span>
          </div>
          <!-- <span>共 <span style="color: #3a8ee6; font-size: 20px">{{totalcount}}</span> 篇</span> -->
        </div>
        <el-row type="flex" align="middle" style="flex-wrap: wrap" :gutter="20" v-for="blog in blogList" :key="blog.id"
          shadow="never" class="blog-content">
          <div @click="getBlogInfo(blog.id)">
            <el-col class="img" :xs="24" :sm="6">
              <el-image v-if="blog.blogPicType == '0'" lazy :src="blog.blogPicLink">
                <div slot="error" class="image-slot">
                  <el-image src="/errorImg.jpg" fit="cover" class="blogPic">></el-image>
                </div>
              </el-image>
              <el-image v-if="blog.blogPicType == '1'" lazy :src="blog.blogPic"></el-image>
            </el-col>
            <el-col :xs="24" :sm="18" style="padding-left: 10px;padding-right: 10px;margin-bottom: 5px;margin-top: -5px;">
              <div>
                <h3><svg-icon icon-class="Topping" v-show="blog.top==1" /> {{blog.title}}</h3>
                <div style="margin-bottom: 10px;">
                    <span style="color: rgba(0, 0, 0, .4);"> {{blog.blogDesc}}</span>
                </div>
                <div style="margin-bottom: 10px;">
                  <el-tag effect="plain" size="mini" v-for="tag in blog.tags" :key="tag.tagId" type="success">
                    {{tag.tagName}}
                  </el-tag>
                </div>
                <div class="blog-info">
                  <div class="user-info">
                    <i class="el-icon-user"></i>
                    <span class="header"> {{blog.createBy}}</span>
                  </div>
                  <div class="blog-date">
                    <i class="el-icon-date"></i>
                    <span> {{blog.createTime}}</span>
                  </div>
                  <div>
                    <i class="el-icon-view"></i>
                    <span> {{blog.views}}</span>
                  </div>
                  <div class="blog-type">
                    <el-tag size="mini" v-for="tag in blog.types" :key="tag.typeId" type="info">
                      {{tag.typeName}}
                    </el-tag>
                  </div>

                </div>
              </div>
            </el-col>
          </div>
        </el-row>
        <pagination v-show="total>0" :total="total" :page.sync="queryParams.pageNum" :limit.sync="queryParams.pageSize" background layout="total, sizes, prev, pager, next, jumper" @pagination="getBlogList"  style="margin-bottom: 30px;float: right;margin-right: 10px;"/>
      </el-card>

    </el-col>
    <el-col :xs="24" :sm="5">
      <el-card style="background-color: rgba(255,255,255,0.9)" class=" right-item">
        <div slot="header" class="attributes">
          <b>分类</b>
        </div>
        <ul class=" blog-type-ul" style="margin-top: 5px;">
          <li class=" blog-type-li" v-for="cmsType in typeList" :key="cmsType.typeId" @click="selectType(cmsType)"
            :class="cmsType.typeId === typeId? 'activeType':''">
            <div style="display: flex;align-items: center">
              <el-image style="width: 28px;height: 28px; border-radius: 50%; margin-right: 10px" lazy
                :src="cmsType.typePicLink" v-show="cmsType.typePicType == '0'">
                <div slot="error" style="width: 28px;height: 28px; border-radius: 50%;">
                  <i class="el-icon-collection" style="margin-left:6px;"></i>
                </div>
              </el-image>
              <el-image style="width: 28px;height: 28px; border-radius: 50%; margin-right: 10px" lazy
                :src="cmsType.typePic" v-show="cmsType.typePicType == '1'">
                <div slot="error" style="width: 28px;height: 28px; border-radius: 50%;">
                  <i class="el-icon-collection" style="margin-left:6px;"></i>
                </div>
              </el-image>
              {{cmsType.typeName}}
            </div>
            <div>{{cmsType.blogNum}}</div>
          </li>
        </ul>
        <div class="more" @click="dealType">
          <i v-if="moreType" class="el-icon-arrow-down"></i>
          <i v-else class="el-icon-arrow-up"></i>
        </div>
      </el-card>
      <el-card style="background-color: rgba(255,255,255,0.9)" class=" right-item">
        <div slot="header" class="attributes">
          <b>标签</b>
        </div>
        <div class="tags">
          <div class=" tag-item" v-for="tag in tagList" :key="tag.tagId" @click="selectTag(tag)"
            :class="tag.tagId === tagId? 'activeTag':''">
            <div class="sjx-outer">
              <div class="sjx-inner"></div>
            </div>
            <div class="tag">
              {{tag.tagName}}
              {{tag.blogNum}}
            </div>
          </div>
        </div>
        <div class="more" @click="dealTag">
          <i v-if="moreTag" class="el-icon-arrow-down"></i>
          <i v-else class="el-icon-arrow-up"></i>
        </div>
      </el-card>
      <el-card style="background-color: rgba(255,255,255,0.9)" class=" right-item">
        <div slot="header" class="attributes">
          <b>最新推荐</b>
        </div>
        <div class=" recommend-blog l-text" v-for="blog in recommendList" :key="blog.id" @click="getBlogInfo(blog.id)">
          <a class="recommend-a">{{blog.title}}</a>
        </div>
      </el-card>
    </el-col>
    <el-col :sm="2" class="hidden-xs-only" style="opacity:0;">右侧占位</el-col>
  </el-row>
</template>

<script>
  import 'element-ui/lib/theme-chalk/display.css';
  import {
    Loading
  } from 'element-ui';
  import {
    cmsListBlog,
    getBlogDetail,
    cmsListByTypeId,
    cmsListByTagId,
    cmsListRecommend,
  } from "@/api/cms/blog";
  export default {
    name: 'cmsIndex',
    data() {
      return {
        totalcount: 100,
        queryInfo: {
          query: '',
          pagenum: 1,
          pagesize: 8
        },
        intro: '',
        blogList: [],
        typeList: [],
        tagList: [],
        fullTypeList: [],
        fullTagList: [],
        recommendList: [],
        selectMethod: '全部博客',
        typeId: -1,
        tagId: -1,
        selected: false,
        moreType: true,
        moreTag: true,
        value: new Date(),
        timer: null,
        start: false,
        screenWidth: document.documentElement.clientWidth, //实时屏幕宽度
        // 查询参数
        queryParams: {
          pageNum: 1,
          pageSize: 10,
          title: null,
          type: 1,
          content: null,
          top: null,
          views: null,
          status: null,
          createBy: null
        },
        // 总条数
        total: 0,
      }
    },
    computed: {
      pagSmall() {
        return this.screenWidth <= 768;
      },
      // 计算分页栏样式
      pagLayout() {
        if (this.screenWidth < 768) {
          return 'prev, pager, next'
        } else {
          return 'total, prev, pager, next, jumper'
        }
      }
    },
    created() {
      window.addEventListener('resize', this.screenAdapter)
    },
    mounted() {
      this.$nextTick(function () {
          // 仅在整个视图都被渲染之后才会运行的代码
          this.getTypeList()
          this.getBlogList();
          this.getTagList()
          this.getRecommendList()
          let str = '这是我的个人博客、会分享关于编程，开发以及其他方面的一些内容，希望可以对您有所帮助...';
          let idx = 0;
          let that = this
          let timer = setTimeout(function fn() {
            // console.log(this.intro)
            that.intro = that.intro + str.substring(idx, idx + 1)
            idx++
            if (idx > str.length) {
              that.intro = ''
              idx = 0
            }
            setTimeout(fn, 200)
          }, 2000)

          this.screenWidth = document.documentElement.clientWidth
        })

    },
    methods: {
      /** 获取博客列表 */
      getBlogList() {
        let loadingInstance = Loading.service({
          target: ".left-item"
        });
        cmsListBlog(this.queryParams).then(response => {
          this.blogList = this.picSrc(response.rows);
          this.total = response.total;
        }).finally(()=>{
          loadingInstance.close();
        });
      },
      //首图地址修改
      picSrc(blogList){
        for (let i = 0; i < blogList.length; i++) {
          let blogInfo = blogList[i];
          if (blogInfo.blogPic.length > 0) {
            blogList[i].blogPic = process.env.VUE_APP_BASE_API + blogInfo.blogPic
          }else{
            blogList[i].blogPic = '/errorImg.jpg'
          }
        };
        return blogList
      },
      // 开始进入主页
      startRead() {
        this.$nextTick(() => {
          document.getElementById('index').scrollIntoView({
            behavior: 'smooth',
            block: 'start',
            // inline: 'nearest'
          });
        })
      },
      compare(property) {
        return function(a, b) {
          let value1 = a[property].length;
          let value2 = b[property].length;
          return value2 - value1;
        }
      },
      // 获取推荐博客列表
      async getRecommendList() {
        cmsListRecommend(this.queryParams).then(response => {
          const {
            data: res
          } = response;
          this.recommendList = response.rows.slice(0, 4);
          this.total = response.total;
        });
      },
      // 获取博客类型列表
      async getTypeList() {
        getBlogDetail(this.$route.query.id).then(response => {
          for (let i = 0; i < response.types.length; i++) {
            let typeInfo = response.types[i];
            if (typeInfo.typePic.length > 0) {
              response.types[i].typePic = process.env.VUE_APP_BASE_API + typeInfo.typePic
            }
          };
          const {
            data: res
          } = response;
          this.fullTypeList = response.types
          this.typeList = response.types.slice(0, 4);
        });
      },
      // 获取博客标签列表
      async getTagList() {
        getBlogDetail(this.$route.query.id).then(response => {
          const {
            data: res
          } = response;
          this.fullTagList = response.tags
          this.tagList = response.tags.slice(0, 6);
        });
      },
      // 跳转到博客详情页
      getBlogInfo(blogId) {
        let routeUrl = this.$router.resolve({
          path: '/cms/main/blog',
          query: {
            id: blogId
          }
        });
        window.open(routeUrl.href, '_blank');
      },
      // 修改当前页码
      handleCurrentChange(newSize) {
        this.queryInfo.pagenum = newSize
        this.getBlogList()
      },
      // 修改当前页大小
      handleSizeChange(newSize) {
        this.queryInfo.pagesize = newSize
      },
      // 按分类筛选博客
      async selectType(cmsType) {
        let loadingInstance = Loading.service({
          target: ".left-item"
        });
        this.typeId = cmsType.typeId
        cmsListByTypeId(this.typeId).then(response => {
          this.blogList = this.picSrc(response.rows);
          this.total = response.total;
          // this.totalcount = res.data.totalElements
          this.selectMethod = '分类: ' + cmsType.typeName
          this.selected = true
        }).finally(()=>{
          loadingInstance.close();
        });
      },
      // 按标签筛选博客
      async selectTag(tag) {
        let loadingInstance = Loading.service({
          target: ".left-item"
        });
        this.tagId = tag.tagId
        cmsListByTagId(this.tagId).then(response => {
          this.blogList = this.picSrc(response.rows);
          this.total = response.total;
          // this.totalcount = res.data.totalElements
          this.selectMethod = '标签: ' + tag.tagName
          this.selected = true
        }).finally(()=>{
          loadingInstance.close();
        });
      },
      // 更新博客列表
      updateBlogList() {
        this.selected = false
        this.typeId = -1
        this.tagId = -1
        this.selectMethod = '全部博客'
        this.getBlogList()
      },
      // 得到所有的标签
      async getFullTagList() {
        this.tagList = this.fullTagList;
      },
      async dealType() {
        if (this.moreType) {
          this.typeList = this.fullTypeList;
        } else {
          this.typeList = this.fullTypeList.slice(0, 4);
        }
        this.moreType = !this.moreType
      },
      async dealTag() {
        if (this.moreTag) {
          await this.getFullTagList()
        } else {
          this.tagList = this.fullTagList.slice(0, 6);
        }
        this.moreTag = !this.moreTag
      },
      // 屏幕尺寸变化的监听函数
      screenAdapter() {
        this.screenWidth = document.documentElement.clientWidth;
      }

    },
  }
</script>

<style scoped>

  .welcome {
    background-color: rgba(0, 0, 0, 0.1);
    border: none;
    height: 90%;
    position: relative;
  }

  .border {
    width: 812px;
    height: 112px;
    position: absolute;
    top: -6px;
    left: -6px;
    border: 3px solid white;
    box-sizing: border-box;
    animation: clipMe 5s linear infinite;
  }

  .tit {
    box-sizing: border-box;
    position: relative;
    width: 800px;
    height: 100px;
    line-height: 100px;
    box-shadow: inset 0 0 0 1px white;
    margin: 40px auto;
    margin-top: 80px;
    color: white;
    text-align: center;
    font-size: 50px;
    font-weight: normal;
    letter-spacing: 10px;
  }

  .intro {
    letter-spacing: 5px;
    line-height: 50px;
    width: 80%;
    margin: 0 auto;
    text-align: center;
    font-weight: normal;
    color: white;
  }

  .down {
    animation: bounce 2s infinite;
    animation-duration: 3s;
    font-size: 25px;
    position: absolute;
    bottom: 5px;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    justify-content: center;
    align-items: center;
    width: 50px;
    height: 50px;
    border-radius: 50%;
    border: 2px solid #fff;
  }

  .down:hover {
    animation: none;
    cursor: pointer;
    box-shadow: 0 0 20px 0 white;
    transition: all .2s;
  }

  .left-item .pagination-container{
    background:rgb(255, 255, 255,0)
  }

  @keyframes clipMe {

    0%,
    100% {
      clip: rect(0px, 806px, 6px, 0px);
    }

    25% {
      clip: rect(0px, 6px, 112px, 0px);
    }

    50% {
      clip: rect(112px, 812px, 112px, 0px);
    }

    75% {
      clip: rect(0px, 812px, 112px, 806px);
    }
  }

  @keyframes bounce {

    0%,
    20%,
    50%,
    80%,
    100% {
      transform: translate(-50%, 0);
    }

    40% {
      transform: translate(-50%, -30px);
    }

    60% {
      transform: translate(-50%, -15px);
    }
  }


  .blog-type-ul {
    padding-left: 10px;
    padding-right: 10px;
    margin-bottom: 0;
    border-radius: 5px;
  }

  .el-pagination {
    padding-bottom: 20px;
  }


  .el-card /deep/ .el-card__body {
    padding: 0;
  }


  .right-item {
    margin-bottom: 20px;
  }

  .blog-type-li:first-child {
    border-top: 1px solid rgba(179, 216, 255, 0.5);
  }

  .blog-type-li {
    border-bottom: 1px solid rgba(179, 216, 255, 0.5);
  }

  .more {
    text-align: center;
    color: #3a8ee6;
    padding: 8px;
  }

  .more:hover {
    cursor: pointer;
    color: #3a8ee6;
  }

  .blog-type-li:hover{
    background-color: rgba(213, 255, 255, 0.3);
    cursor: pointer;
  }

  .activeType {
    background-color: rgba(58, 142, 230, 0.3);
    cursor: pointer;
  }

  .tags {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    margin: 15px 13px 0;
    border-bottom: 1px solid rgba(179, 216, 255, 0.5);

  }

  .tag-item {
    display: flex;
    justify-content: space-around;
    align-items: center;
    margin-left: 5px;
    margin-right: 5px;
    margin-bottom: 10px;
    box-sizing: border-box;
  }

  .tag {
    background-color: #ecf5ff;
    box-sizing: border-box;
    display: inline-block;
    height: 22px;
    padding: 0 10px;
    line-height: 22px;
    font-size: 10px;
    color: #409eff;
    border-radius: 4px;
    white-space: nowrap;
    border: 1px solid #409eff;
    transition: .2s;
  }

  .sjx-outer {
    width: 0;
    height: 0;
    border-top: 6px solid transparent;
    border-bottom: 6px solid transparent;
    border-right: 6px solid #409eff;
    position: relative;
    transition: .2s;
  }

  .sjx-inner {
    border-top: 6px solid transparent;
    border-bottom: 6px solid transparent;
    border-right: 6px solid #ecf5ff;
    top: -6px;
    left: 1px;
    position: absolute;
    transition: .2s;
  }

  .tag-item:hover,
  .activeTag {
    box-sizing: border-box;
  }

  .tag {
    color: white;
    background-color: #409eff;
    cursor: pointer;
  }

  .sjx-inner {
    border-right: 6px solid #409eff;
  }

  .blog-type-li {
    display: flex;
    justify-content: space-between;
    align-items: center;
    line-height: 40px;
  }

  .recommend-blog {
    overflow: hidden;
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-line-clamp: 1;
    -webkit-box-orient: vertical;
    padding-left: 10px;
    padding-right: 10px;
    margin-bottom: 0;
    border-radius: 5px;
  }

  .recommend-a {
    border-bottom: 1px solid rgba(34, 36, 38, .15);
    line-height: 40px;
    display: block;
    text-decoration: none;
    color: black;
  }

  .recommend-a:hover {
    color: #3a8ee6;
  }

  .total {
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-size: larger;
    font-weight: bold;
  }

  .titleIndex {
    display: flex;
    align-items: center;
  }

  .el-icon-back {
    font-weight: bolder;
    color: #3a8ee6;
    margin-right: 10px;
  }

  .el-icon-back:hover {
    cursor: pointer;
  }

  .blog-content:hover {
    border-left: 5px solid #3a8ee6;
    border-right: 5px solid #3a8ee6;
    background-color: rgba(58, 142, 230, 0.3);
    cursor: pointer;
  }

  .blog-content {
    padding: 10px;
    height: auto;
    border-bottom: 1px solid rgb(199, 163, 92);
    /*border-bottom: 1px solid rgba(34, 36, 38, .15);*/
    transition: .3s;
  }

  .el-image {
    border-radius: 5px;
    box-sizing: border-box;
    flex-shrink: 0;
  }

  .blog-info {
    display: flex;
    align-items: center;
    color: rgba(0, 0, 0, .4);
    font-size: 10px;
  }

  .user-info {
    display: flex;
    justify-content: space-around;
    align-items: center;
    margin-right: 15px;
    float: left;
  }

  .header {
    text-decoration: none;
    color: #3a8ee6;
    font-weight: bold;
  }

  .blog-date {
    float: right;
    margin-right: 15px;
  }

  .blog-type {
    float: right;
    margin-left: auto;
  }

  .blog-tag {
    float: right;
    margin-left: auto;
  }
  @media screen and (max-width: 768px) {
    .blog-date {
      display: none;
    }

    .welcome {
      width: 100%;
    }

    .border {
      display: none;
    }

    .tit {
      font-size: 2rem;
      width: 100%;
      line-height: 50px;
      letter-spacing: 2px;
      height: auto;
    }

    .intro {
      font-size: 1rem;
      line-height: 30px;
    }

    .el-pagination {
      width: 100%;
    }
  }
</style>
