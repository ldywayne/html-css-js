<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="标题" prop="title">
        <el-input v-model="queryParams.title" placeholder="请输入标题" clearable size="small"
          @keyup.enter.native="handleQuery" />
      </el-form-item>
      <el-form-item label="状态" prop="status">
        <el-select v-model="queryParams.status" placeholder="请选择状态" clearable size="small">
          <el-option v-for="dict in dict.type.cms_blog_status" :key="dict.value" :label="dict.label"
            :value="dict.value" />
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button type="primary" plain icon="el-icon-plus" size="mini" @click="handleAdd"
          v-hasPermi="['cms:blog:add']">新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button type="success" plain icon="el-icon-edit" size="mini" :disabled="single" @click="handleUpdate"
          v-hasPermi="['cms:blog:edit']">修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button type="danger" plain icon="el-icon-delete" size="mini" :disabled="multiple" @click="handleDelete"
          v-hasPermi="['cms:blog:remove']">删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button type="warning" plain icon="el-icon-download" size="mini" @click="handleExport"
          v-hasPermi="['cms:blog:export']">导出</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="blogList" @selection-change="handleSelectionChange"
      :row-class-name="tableRowClassName">
      <el-table-column type="selection" width="55" align="center" />
      <!-- <el-table-column label="ID" align="center" prop="id" /> -->
      <el-table-column label="首图预览" align="center" prop="blogPic" >
        <template slot-scope="scope">
          <el-image
            v-if="scope.row.blogPicType == '0'"
            style="width: 120px;height: 60px;"
            :src="scope.row.blogPicLink"
            lazy
            :preview-src-list="[scope.row.blogPicLink]">
              <div slot="error" class="image-slot">
                <el-image src="/errorImg.jpg" fit="cover" class="blogPic"></el-image>
              </div>
          </el-image>
          <el-image
            v-if="scope.row.blogPicType == '1'"
            style="width: 120px;height: 60px;"
            :src="scope.row.blogPic"
            lazy
            :preview-src-list="[scope.row.blogPic]">
          </el-image>
        </template>
      </el-table-column>
      <el-table-column label="标题" align="center" prop="title" />
      <!-- <el-table-column label="内容" align="center" prop="content" /> -->
      <!-- <el-table-column label="置顶" align="center" prop="top" /> -->
      <el-table-column label="分类" align="center" prop="types">
        <template slot-scope="scope">
          <el-tag size="mini" v-for="tag in scope.row.types" :key="tag.typeId" type="info">{{tag.typeName}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="标签" align="center" prop="tags">
        <template slot-scope="scope">
          <el-tag effect="plain" size="mini" v-for="tag in scope.row.tags" :key="tag.tagId" type="success">
            {{tag.tagName}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="阅读量" align="center" prop="views" />
      <el-table-column label="状态" align="center" prop="status">
        <template slot-scope="scope">
          <dict-tag :options="dict.type.cms_blog_status" :value="scope.row.status" />
        </template>
      </el-table-column>
      <el-table-column label="创建者" align="center" prop="createBy" />
      <el-table-column label="创建时间" align="center" prop="createTime" width="100">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.createTime, '{y}-{m}-{d} {h}:{i}:{s}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button size="mini" type="text" icon="el-icon-edit" @click="handleUpdate(scope.row)"
            v-hasPermi="['cms:blog:edit']">修改</el-button>
          <el-button size="mini" type="text" icon="el-icon-delete" @click="handleDelete(scope.row)"
            v-hasPermi="['cms:blog:remove']">删除</el-button>
            <el-button size="mini" type="text" icon="el-icon-folder-opened" @click="blogFiles(scope.row)"
              v-hasPermi="['cms:blog:edit']">附件管理</el-button>
          <!-- <el-button size="mini" type="text" icon="el-icon-folder-opened" @click="fileList(scope.row)"
            v-hasPermi="['cms:blog:edit']">资源列表</el-button> -->
        </template>
      </el-table-column>
    </el-table>

    <pagination v-show="total>0" :total="total" :page.sync="queryParams.pageNum" :limit.sync="queryParams.pageSize"
      @pagination="getList" />

    <!-- 添加或修改文章管理对话框 -->
    <el-dialog :title="title" :visible.sync="open" :before-close="cancel" width="1200px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="标题" prop="title">
          <el-input v-model="form.title" placeholder="请输入标题" />
        </el-form-item>
        <el-row>
          <el-col :span="8">
            <el-form-item label="首图">
              <el-radio-group v-model="form.blogPicType">
                <el-radio-button label="0">地址</el-radio-button>
                <el-radio-button label="1">上传</el-radio-button>
              </el-radio-group>
              <div v-show="form.blogPicType == '0'" class="tabBlock">
                <el-input v-model="form.blogPicLink" placeholder="请输入图片地址 https://" style="margin-bottom: 10px;" />
                <el-image :src="form.blogPicLink" :preview-src-list="[form.blogPicLink]" fit="cover" class="blogPic" >
                  <div slot="error" class="image-slot">
                    <el-image src="/errorImg.jpg" fit="cover" class="blogPic"></el-image>
                  </div>
                </el-image>
              </div>
              <div v-show="form.blogPicType == '1'" class="tabBlock">
                <imageUpload v-model="form.blogPic" :limit="1" />
              </div>
            </el-form-item>
          </el-col>
          <el-col :span="16">
            <el-form-item label="简介">
              <el-input type="textarea" v-model="form.blogDesc" :autosize="{ minRows: 7, maxRows: 7}" maxlength="50" show-word-limit placeholder="请输入简介" />
            </el-form-item>
          </el-col>
        </el-row>
        <el-form-item label="内容">
          <!-- 图片用base64存储,url方式移动端会显示异常 -->
          <el-row style="margin-bottom: 20px;">
            <el-col align="right">
              <span v-show="form.contentType ==='2'" style="color: #E6A23C;margin-right: 20px;">Markdown编辑器保存后会覆盖其他富文本编辑器内容</span>
              编辑器：
              <el-select v-model="form.contentType" placeholder="请选择">
                <el-option key="1" label="Quill富文本编辑器" value="1" />
                <el-option key="2" label="CherryMarkdown (推荐)" value="2" />
                <el-option key="3" label="Tinymce富文本编辑器" value="3" />
              </el-select>
            </el-col>
          </el-row>
          <el-row>
            <el-col>
              <cmsEditor v-if="form.contentType ==='1'" v-model="form.content" @getFileId="getFileId" type="base64" :min-height="2000" />
              <CherryMarkdown ref="CherryMarkdown" v-if="form.contentType ==='2'" :height='2000' v-model='form.contentMarkdown' ></CherryMarkdown>
              <Tinymce v-if="form.contentType ==='3'" :height='2000' v-model='form.content'></Tinymce>
            </el-col>
          </el-row>
        </el-form-item>
        <el-form-item label="标签">
          <el-checkbox-group v-model="form.tagIds">
            <el-checkbox v-for="item in tagOptions" :label="item.tagId" :key="item.tagId" :value="item.tagId">
              {{item.tagName}}
            </el-checkbox>
          </el-checkbox-group>
        </el-form-item>
        <el-row>
          <el-col :span="7">
            <el-form-item label="分类">
              <el-select v-model="form.typeIds" multiple placeholder="请选择" filterable clearable>
                <el-option v-for="item in typeOptions" :key="item.typeId" :label="item.typeName" :value="item.typeId">
                </el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="17">
            <el-form-item label="置顶">
              <el-checkbox v-model="top"></el-checkbox>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="releaseForm">发 布</el-button>
        <el-button type="info" @click="saveForm">暂 存</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </el-dialog>

    <!-- 资源列表对话框 -->
    <el-dialog :title="title" :visible.sync="fileListOpen" width="1000px" append-to-body>
      <el-table class="file-list" :data="fileInfoList">
        <el-table-column type="selection" width="55" align="center" />
        <!-- <el-table-column label="文件主键id" align="center" prop="fileId" /> -->
        <el-table-column label="图片预览" align="center" prop="pic">
          <template slot-scope="scope">
            <el-image style="width: 120px;height: 60px;" :src="scope.row.pic" lazy :preview-src-list="[scope.row.pic]">
            </el-image>
          </template>
        </el-table-column>
        <el-table-column label="文件名称" align="center" prop="fileOriginName" />
        <el-table-column label="文件类型" align="center" prop="fileSuffix" />
        <el-table-column label="文件大小" align="center" prop="fileSizeInfo" />
        <!-- <el-table-column label="存储文件名称" align="center" prop="fileObjectName" /> -->
        <!-- <el-table-column label="存储路径" align="center" prop="filePath" /> -->
        <!-- <el-table-column label="创建者" align="center" prop="createBy" /> -->
        <!-- <el-table-column label="创建时间" align="center" prop="createTime" width="100">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.createTime, '{y}-{m}-{d} {h}:{i}:{s}') }}</span>
          </template>
        </el-table-column> -->
        <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
          <template slot-scope="scope">
            <el-button size="mini" type="text" icon="el-icon-download" @click="handleDownload(scope.row)">下载</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-dialog>

    <!-- 附件管理对话框 -->
    <el-dialog title="附件管理" :visible.sync="blogFilesOpen" :before-close="cancel" width="1200px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-position="top" label-width="80px">
        <el-form-item>
          <el-row>
            <el-button size="mini" type="primary" @click="addFiles">添加文件</el-button>
            <el-table :data="form.blogFilesNew" :border="true" style="width: 99.99%;">
              <el-table-column align="center" min-width="20%" prop="pic" label="附件">
                <template slot-scope="scope">
                  <filesUpload v-model="scope.row.fileId" @handleFilesSuccess="filesSuccess" :is-show-tip="false" />
                </template>
              </el-table-column>
              <el-table-column align="center" min-width="20%" prop="remark" label="文件信息">
                <template slot-scope="scope">
                  <el-row>
                    <el-col :span="6"><div class="blogFilesInfoName">名称：</div></el-col>
                    <el-col :span="18"><el-input v-model="scope.row.fileOriginName" disabled/></el-col>
                  </el-row>
                  <el-row style="margin-top: 4px;">
                    <el-col :span="6"><div class="blogFilesInfoName">大小：</div></el-col>
                    <el-col :span="18"><el-input v-model="scope.row.fileSize" disabled/></el-col>
                  </el-row>
                  <el-row style="margin-top: 4px;">
                    <el-col :span="6"><div class="blogFilesInfoName">类型：</div></el-col>
                    <el-col :span="18"><el-input v-model="scope.row.fileSuffix" disabled/></el-col>
                  </el-row>
                </template>
              </el-table-column>
              <el-table-column align="center" min-width="40%" prop="remark" label="备注">
                <template slot-scope="scope">
                  <el-input v-model="scope.row.remark" type="textarea" :rows="6" size="small" />
                </template>
              </el-table-column>
              <el-table-column align="center" min-width="20%" label="操作">
                <template slot-scope="scope">
                  <el-button v-show="scope.row.fileId!==''" size="mini" plain @click="handleDownload(scope.row)">下载</el-button>
                  <el-button size="mini" type="danger" plain @click="delFiles(scope.$index, scope.row)">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-row></el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="saveBlogFiles">保 存</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </el-dialog>

  </div>
</template>

<script>
  import filesUpload from './components/filesUpload'
  import CherryMarkdown from '@/components/CherryMarkdown'
  import Tinymce from '@/components/Tinymce'
  import {
    listBlog,
    getBlog,
    delBlog,
    addBlog,
    updateBlog,
    cancelBlog
  } from "@/api/cms/blog";
  import {
    delFileInfo
  } from "@/api/cms/fileInfo";
  import {
    addFileBlogInfo,
    delFileBlogInfo,
    getFileList
  } from "@/api/cms/fileBlogInfo";
  import {
    Loading
  } from 'element-ui';

  export default {
    name: "Blog",
    dicts: ['cms_blog_status'],
    components: {
      filesUpload,
      CherryMarkdown,
      Tinymce
    },
    data() {
      return {
        // 遮罩层
        loading: true,
        // 选中数组
        ids: [],
        names: [],
        // 非单个禁用
        single: true,
        // 非多个禁用
        multiple: true,
        // 显示搜索条件
        showSearch: true,
        // 总条数
        total: 0,
        // 文章管理表格数据
        blogList: [],
        // 资源列表表格数据
        fileInfoList: [],
        // 弹出层标题
        title: "",
        // 是否显示弹出层
        open: false,
        fileListOpen: false,
        blogFilesOpen: false,
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
        // 表单参数
        form: {},
        top: false,
        // 表单校验
        rules: {
          title: [{
            required: true,
            message: "标题不能为空",
            trigger: "blur"
          }],
          type: [{
            required: true,
            message: "类型不能为空",
            trigger: "change"
          }],
        },
        fileIds: [],
        // 类型选项
        typeOptions: [],
        // 标签选项
        tagOptions: [],
      };
    },
    created() {
      this.getList();
    },
    methods: {
      /** 查询文章管理列表 */
      getList() {
        this.loading = true;
        listBlog(this.queryParams).then(response => {
          for (let i = 0; i < response.rows.length; i++) {
            let blogInfo = response.rows[i];
            if (blogInfo.blogPic.length > 0) {
              response.rows[i].blogPic = process.env.VUE_APP_BASE_API + blogInfo.blogPic
            }else{
              response.rows[i].blogPic = '/errorImg.jpg'
            }
            if (blogInfo.blogPicLink.length == 0) {
              response.rows[i].blogPicLink = '/errorImg.jpg'
            }
          };
          this.blogList = response.rows;
          this.total = response.total;
          this.loading = false;
        });
      },
      // 取消按钮
      cancel() {
        this.$confirm('是否放弃此次编辑？', '系统提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          let fileids = this.fileIds;
          if (fileids.length > 0) {
            delFileInfo(fileids);
          };
          this.fileIds.length = 0;
          cancelBlog(this.form).then(response => {});
          this.top = false;
          this.open = false;
          this.blogFilesOpen = false;
          this.reset();
        }).catch(() => {});
      },
      // 表单重置
      reset() {
        this.form = {
          id: null,
          createBy: null,
          createTime: null,
          updateBy: null,
          updateTime: null,
          title: null,
          type: 1,
          content: null,
          top: "0",
          views: null,
          status: "0",
          blogDesc: null,
          blogFiles: null,
          blogPicType: "0",
          blogPic: null,
          blogPicLink: null,
          tagIds: [],
          typeIds: [],
          blogFilesNew: [],
          contentType: "1",
          contentMarkdown: null
        };
        this.resetForm("form");
      },
      /** 搜索按钮操作 */
      handleQuery() {
        this.queryParams.pageNum = 1;
        this.getList();
      },
      /** 重置按钮操作 */
      resetQuery() {
        this.resetForm("queryForm");
        this.handleQuery();
      },
      // 多选框选中数据
      handleSelectionChange(selection) {
        this.ids = selection.map(item => item.id)
        this.names = selection.map(item => item.title)
        this.single = selection.length !== 1
        this.multiple = !selection.length
      },
      /** 新增按钮操作 */
      handleAdd() {
        getBlog().then(response => {
          this.typeOptions = response.types;
          this.tagOptions = response.tags;
          this.reset();
          this.open = true;
          this.title = "添加文章";
        });
      },
      /** 修改按钮操作 */
      handleUpdate(row) {
        this.reset();
        const id = row.id || this.ids
        getBlog(id).then(response => {
          this.typeOptions = response.types;
          this.tagOptions = response.tags;
          this.form = response.data;
          if (this.form.top == 1) {
            this.top = true;
          };
          this.open = true;
          this.title = "修改文章";
        });
      },
      /** 发布按钮 */
      releaseForm() {
        this.$refs["form"].validate(valid => {
          if (valid) {
            this.$modal.confirm('是否确认发布？').then(()=>{
              this.form.type = 1;
              this.form.status = 1;
              if (this.top) {
                this.form.top = 1;
              } else {
                this.form.top = 0;
              }
              if (this.form.contentType === '2'){
                this.setFormContent()
              }
              if (this.form.id != null) {
                updateBlog(this.form).then(response => {
                  if (this.fileIds.length > 0) {
                    let fileBlogInfo = {
                      blogId: this.form.id,
                      fileIds: this.fileIds
                    };
                    addFileBlogInfo(fileBlogInfo).then(response => {});
                  }
                  this.$modal.msgSuccess("发布成功");
                  this.fileIds.length = 0;
                  this.open = false;
                  this.getList();
                });
              } else {
                addBlog(this.form).then(response => {
                  if (this.fileIds.length > 0) {
                    let fileBlogInfo = {
                      blogId: response.data,
                      fileIds: this.fileIds
                    };
                    addFileBlogInfo(fileBlogInfo).then(response => {});
                  }
                  this.$modal.msgSuccess("发布成功");
                  this.fileIds.length = 0;
                  this.open = false;
                  this.getList();
                });
              }
            }).catch(() => {})
          }
        });
      },
      /** 暂存按钮 */
      saveForm() {
        this.$refs["form"].validate(valid => {
          if (valid) {
            this.$modal.confirm('是否确认暂存？').then(()=>{
              this.form.type = 1;
              this.form.status = 0;
              if (this.top) {
                this.form.top = 1;
              } else {
                this.form.top = 0;
              }
              if (this.form.contentType === '2'){
                this.setFormContent()
              }
              if (this.form.id != null) {
                updateBlog(this.form).then(response => {
                  if (this.fileIds.length > 0) {
                    let fileBlogInfo = {
                      blogId: this.form.id,
                      fileIds: this.fileIds
                    };
                    addFileBlogInfo(fileBlogInfo).then(response => {});
                  }
                  this.$modal.msgSuccess("暂存成功");
                  this.fileIds.length = 0;
                  this.open = false;
                  this.getList();
                });
              } else {
                addBlog(this.form).then(response => {
                  if (this.fileIds.length > 0) {
                    let fileBlogInfo = {
                      blogId: response.data,
                      fileIds: this.fileIds
                    };
                    addFileBlogInfo(fileBlogInfo).then(response => {});
                  }
                  this.$modal.msgSuccess("暂存成功");
                  this.fileIds.length = 0;
                  this.open = false;
                  this.getList();
                });
              }
            }).catch(() => {})
          }
        });
      },
      /** 删除按钮操作 */
      handleDelete(row) {
        const ids = row.id || this.ids;
        let name = row.title || this.names;
        this.$modal.confirm('是否确认删除"' + name + '"？').then(function() {
          delFileBlogInfo(ids).then().then(response => {});
          return delBlog(ids);
        }).then(() => {
          this.getList();
          this.$modal.msgSuccess("删除成功");
        }).catch(() => {});
      },
      /** 导出按钮操作 */
      handleExport() {
        this.download('cms/blog/export', {
          ...this.queryParams
        }, `blog_${new Date().getTime()}.xlsx`)
      },
      getFileId(data) {
        this.fileIds.push(data);
      },
      /** 资源列表按钮操作 */
      fileList(row) {
        let loadingInstance = Loading.service({
          target: ".file-list"
        });
        this.reset();
        const blogId = row.id || this.ids
        getFileList(blogId).then(response => {
          for (let i = 0; i < response.data.length; i++) {
            let fileInfo = response.data[i];
            switch (fileInfo.fileSuffix) {
              case 'png':
              case 'jpg':
              case 'jpeg':
              case 'bmp':
              case 'gif':
                response.data[i].pic = process.env.VUE_APP_BASE_API + fileInfo.filePath;
                break;
              default:
                response.data[i].pic = image.bg1;
                break;
            };
          };
          this.fileInfoList = response.data;
          this.fileListOpen = true;
          this.title = "资源列表";
          setTimeout(() => {
            loadingInstance.close();
          }, 100);
        });
      },
      /** 附件管理按钮操作 */
      blogFiles(row) {
        this.reset();
        const id = row.id || this.ids
        getBlog(id).then(response => {
          this.typeOptions = response.types;
          this.tagOptions = response.tags;
          this.form = response.data;
          this.form.blogFilesNew = []
          if (response.data.blogFiles !== null) {
            this.form.blogFilesNew = JSON.parse(response.data.blogFiles)
          }
          if (this.form.top == 1) {
            this.top = true;
          };
          this.blogFilesOpen = true;
        });
      },
      // 附件管理添加按钮
      addFiles() {
        this.form.blogFilesNew.push(
          {
            id: this.uuid(),
            fileId: '',
            fileOriginName: '',
            fileSuffix: '',
            fileSize: '',
            filePath: '',
            remark: ''
          })
      },
      delFiles(index, row) {
        this.$confirm('确认删除吗?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          // 点击确定的操作(调用接口)
          var hasmembers = this.form.blogFilesNew
          for (var i = 0; i < hasmembers.length; i++) {
            if (row.id === hasmembers[i].id) {
              this.form.blogFilesNew.splice(i, 1)
            }
          }
        }).catch(() => {
          // 点取消的提示
          return
        })
      },
      // 生成uuid
      uuid() {
        var d = new Date().getTime()
        if (window.performance && typeof window.performance.now === 'function') {
          d += performance.now() // use high-precision timer if available
        }
        var uuid = 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
          var r = (d + Math.random() * 16) % 16 | 0 // d是随机种子
          d = Math.floor(d / 16)
          return (c === 'x' ? r : (r & 0x3 | 0x8)).toString(16)
        })
        return uuid
      },
      filesSuccess(value){
        this.form.blogFilesNew.forEach(item => {
          if(item.fileId===value.fileId){
            item.fileOriginName = value.fileOriginName
            item.fileSuffix = value.fileSuffix
            item.fileSize = value.fileSize
            item.filePath = value.filePath
          }
        });
      },
      //保存文件
      saveBlogFiles(){
        if (this.form.blogFilesNew.length > 0) {
          for (let i = 0; i < this.form.blogFilesNew.length; i++) {
            const fileInfo = this.form.blogFilesNew[i]
            if (fileInfo.fileId === '') {
              this.$message.warning('请添加文件或删除空行！')
              return false
            } else if (fileInfo.remark === '') {
              this.$message.warning('请添加文件备注！')
              return false
            }
          }
        }
        this.form.blogFiles = JSON.stringify(this.form.blogFilesNew)
        updateBlog(this.form).then(response => {
          this.$modal.msgSuccess("保存成功");
          this.blogFilesOpen = false;
          this.getList();
        });
      },
      // 文件下载处理
      handleDownload(row) {
        var name = row.fileOriginName;
        var url = row.filePath;
        var suffix = url.substring(url.lastIndexOf("."), url.length);
        const a = document.createElement('a')
        a.setAttribute('download', name)
        a.setAttribute('target', '_blank')
        a.setAttribute('href', process.env.VUE_APP_BASE_API + url)
        a.click()
      },
      tableRowClassName({
        row,
        rowIndex
      }) {
        if (row.top == 1) {
          return 'warning-row';
        }
        return '';
      },
      setFormContent(){
        this.form.content = this.$refs.CherryMarkdown.getCherryHtml()
      },
    }
  };
</script>

<style scoped lang="scss">
  .el-tag+.el-tag {
    margin-left: 10px;
  }
  .el-table .warning-row {
      background: #f8f8f9;
    }
  .blogFilesInfoName {
    text-align: center;
    padding-top: 5px;
  }
  .tabBlock {
    height: 180px;
    margin-top: 20px;
  }
  .blogPic {
    width: 200px;
    height: 100px;
  }
</style>
