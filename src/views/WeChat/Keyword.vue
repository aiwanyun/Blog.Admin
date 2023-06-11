<template>
  <section>
    <!--工具条-->
    <el-row :gutter="10">
      <el-col :span="1.5">
        <el-select v-model="selectWeChat" placeholder="请选择要操作的公众号" @change="handleSelectWeChat">
          <el-option v-for="item in wechats" :key="item.value" :label="item.label" :value="item.value">
            <span style="float: left">{{ item.label }}</span>
            <span style="float: right; color: #8492a6; font-size: 13px">{{ item.value }}</span>
          </el-option>
        </el-select>
      </el-col>
      <el-col :span="1.5">
        <el-input v-model="filters.name" auto-complete="off" placeholder="请输入搜索关键词" style="width: 150px;"></el-input>
      </el-col>
      <el-col :span="1.5">
        <el-button type="primary" :disabled="selectWeChat == ''" @click="handleSearch">查询</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button type="primary" :disabled="selectWeChat == ''" @click="handleAdd">新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button type="primary" :disabled="selectWeChat == ''" @click="handleEdit">编辑</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button type="danger" :disabled="selectWeChat == ''" @click="handleDel">删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button type="danger" :disabled="selectWeChat == '' || sels.length === 0" @click="batchRemove">批量删除</el-button>
      </el-col>
    </el-row>
    <el-form :inline="true" @submit.native.prevent>
      <el-form-item>

      </el-form-item>
      <el-form-item>




      </el-form-item>
    </el-form>

    <!--列表-->
    <el-table :data="tableData" highlight-current-row v-loading="listLoading" @selection-change="selsChange"
      @current-change="selectCurrentRow" style="width: 100%;">
      <el-table-column type="selection" width="60"></el-table-column>
      <el-table-column type="index" width="80"></el-table-column>
      <el-table-column prop="key" label="触发关键词" width show-overflow-tooltip></el-table-column>
      <el-table-column prop="media_type" label="关键词类型" width show-overflow-tooltip></el-table-column>
      <el-table-column prop="media_desc" label="关键词回复-文本" width show-overflow-tooltip></el-table-column>
      <el-table-column prop="title" label="视频标题" width show-overflow-tooltip></el-table-column>
      <el-table-column prop="description" label="视频描述" width show-overflow-tooltip></el-table-column>
      <el-table-column prop="url" label="关键词回复-媒体地址" width show-overflow-tooltip></el-table-column>
      <el-table-column prop="media_id" label="关键词回复-媒体id" width show-overflow-tooltip></el-table-column>
    </el-table>
    <!--工具条-->
    <el-col :span="24" class="toolbar">
      <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="page.pageSize"
        :total="total" :current-page.sync="page.pageIndex" style="float:right;"></el-pagination>
    </el-col>

    <!--编辑界面-->
    <el-dialog :title="editType" :visible.sync="editFormVisible" :close-on-click-modal="false" append-to-body>
      <el-form :model="editForm" label-width="200px" :rules="rules" ref="editForm" label-position="top">
        <el-form-item label="触发关键词" prop="key">
          <el-input v-model="editForm.key" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="媒体类型" prop="media_type">
          <el-select v-model="editForm.media_type" placeholder="请选择">
            <el-option label="文本" value="text"></el-option>
            <el-option label="图片" value="image"></el-option>
            <el-option label="语音" value="voice"></el-option>
            <el-option label="视频" value="video"></el-option>
            <el-option label="缩略图" value="thumb"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item v-if="editForm.media_type != 'text'" label="回复媒体" prop="media_id">
          <el-input v-show="false" v-model="editForm.media_id" type="textarea" auto-complete="off"
            :disabled="true"></el-input>
          <el-row v-if="editForm.media_type == 'video'">
            <el-col>
              视频标题:<el-input style="width:calc(100% - 100px);margin-left: 5px;" v-model="editForm.title"
                auto-complete="off"></el-input>
            </el-col>
            <el-col style="margin-top: 5px;margin-bottom: 5px;">
              视频描述:<el-input style="width:calc(100% - 100px);margin-left: 5px;" v-model="editForm.description"
                auto-complete="off"></el-input>
            </el-col>
          </el-row>
          <el-row :gutter="10">

            <el-col :span="1.5">
              <el-upload class="upload-demo" action="#" :before-upload="beforeUpload">
                <el-link type="primary">新上传</el-link>
              </el-upload>
            </el-col>
            <el-col :span="1.5">
              <el-link type="primary" @click="handleMedia">选择</el-link>
            </el-col>
            <el-col :span="1.5">
              <el-link v-show="editForm.url" type="primary">
                <a target="_blank" :href="editForm.url" rel="noreferrer noopener nofollow">查看</a>
              </el-link>
            </el-col>
          </el-row>
          <el-row :gutter="10">
            <el-col>
              <el-input v-model="editForm.url" :rows="5" type="textarea" auto-complete="off" :disabled="true"></el-input>
            </el-col>
          </el-row>
        </el-form-item>
        <el-form-item v-if="editForm.media_type == 'text'" label="回复文本" prop="media_desc">
          <el-input type="textarea" :rows="5" v-model="editForm.media_desc" auto-complete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="editFormVisible = false">取消</el-button>
        <el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
      </div>
    </el-dialog>


    <el-dialog title="素材选择" :visible.sync="visibleMedia" :close-on-click-modal="false" append-to-body>

      <el-table :data="mediaList" style="width: 100%" highlight-current-row @current-change="handleSelectMedia">
        <el-table-column prop="name" label="名称" width="300" show-overflow-tooltip>
        </el-table-column>
        <el-table-column prop="url" label="地址" min-width="100" show-overflow-tooltip>
        </el-table-column>
        <el-table-column prop="description" label="描述" min-width="100" show-overflow-tooltip>
        </el-table-column>
        <el-table-column prop="media_id" label="媒体id" width="100" show-overflow-tooltip>
        </el-table-column>
      </el-table>
      <el-col :span="24" class="toolbar">
        <el-pagination layout="prev, pager, next" @current-change="handleCurrentMedia" :page-size="10" :total="mediaTotal"
          :current-page.sync="curMediaPage" style="float:right;"></el-pagination>
      </el-col>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="visibleMedia = false">取消</el-button>
        <el-button type="primary" @click.native="editSubmitMedia" :loading="editLoading">确定</el-button>
      </div>
    </el-dialog>
  </section>
</template>

<script>
import {
  getWeChatAccount,
  getWeChatKeyword,
  removeWeChatKeyword,
  addWeWeChatKeyword,
  updateWeChatKeyword,
  batchDeleteWeChatKeyword,
  UpdateWeChatFile,
  GetWeChatMediaList
} from "../../api/api";
export default {
  components: {},
  data() {
    return {
      filters: {
        name: ""
      },
      listLoading: false,
      tableData: [],
      sels: [],
      currentRow: null,
      page: {
        pageSize: 10,
        pageIndex: 1,
      },
      total: 0,
      editFormVisible: false,
      editLoading: false,
      editType: "add", //默认新增类型
      editForm: {},
      statusList: [{ LinkUrl: '激活', value: true }, { LinkUrl: '禁用', value: false }],
      selectWeChat: '',
      wechats: [], //微信公众号列表
      curMediaPage: 1,
      mediaTotal: 1,
      mediaList: [],
      visibleMedia: false,
      curSelectMedia: {},
      rules: {
        key: [
          { required: true, message: '请输入关键词', trigger: 'blur' },
        ],
        media_type: [
          { required: true, message: '请选择媒体类型', trigger: 'blur' },
        ],
        media_id: [
          { required: true, message: '请选择回复媒体', trigger: 'blur' },
        ],
        media_desc: [
          { required: true, message: '请输入回复文本', trigger: 'blur' },
        ],
      }
    };
  },
  created() {
    // this.getWeChatKeyword();
    this.getWeChats();
  },
  methods: {
    handleCurrentMedia(index) {
      this.curMediaPage = index;
      this.GetWeChatMediaList()
    },
    handleSelectMedia(row) {
      this.curSelectMedia = row
    },
    editSubmitMedia() {
      if (!this.curSelectMedia.media_id) {
        this.$message.error("请选一个素材");
        return false;
      }

      this.editForm.url = this.curSelectMedia.url
      this.editForm.media_id = this.curSelectMedia.media_id
      if (this.editForm.media_type == 'video') {
        this.editForm.title = this.curSelectMedia.name
        this.editForm.description = this.curSelectMedia.description
      }
      this.visibleMedia = false
    },
    handleMedia() {
      this.curMediaPage = 1
      this.GetWeChatMediaList()
    },
    getWeChats() {
      getWeChatAccount().then(res => {
        this.wechats = [];
        if (res.data.success) {
          res.data.response.data.forEach(element => {
            this.wechats.push({
              value: element.publicAccount,
              label: element.publicNick
            });
          });
        } else {
          this.$message.error(res.data.msg);
        }
      });
    },
    GetWeChatMediaList() {
      this.mediaList = []
      GetWeChatMediaList({ id: this.selectWeChat, type: this.editForm.media_type, page: this.curMediaPage })
        .then(res => {

          if (res.data.success) {
            this.mediaTotal = res.data.response.total_count
            this.mediaList = res.data.response.item
            this.visibleMedia = true
            this.curSelectMedia = {}
          }
        })
    },
    beforeUpload(file) {
      if (!this.editForm.media_type) {
        this.$message.error("请选择媒体类型再上传");
        return false;
      }
      if (!this.editForm.publicAccount) {
        this.$message.error("请选择要操作的公众号");
        return false;
      }
      if (this.editForm.media_type == 'video') {
        if (!this.editForm.title || !this.editForm.description) {
          this.$message.error("请先填写视频的标题和描述");
          return false;
        }
      }
      // const isJPG = file.type === 'image/jpeg';
      // const isLt2M = file.size / 1024 / 1024 < 2;
      var formData = new FormData();
      formData.append("media", file);

      var des = { "title": this.editForm.title, "introduction": this.editForm.description }
      formData.append("description", JSON.stringify(des));
      UpdateWeChatFile(this.editForm.publicAccount, this.editForm.media_type, formData).then(res => {
        if (res.data.success) {
          this.$set(this.editForm, "media_id", res.data.response.media_id)
          this.$set(this.editForm, "url", res.data.response.url.replace("http://", "https://"))
        }
      })
      return false;
    },
    selectCurrentRow(val) {
      this.currentRow = val;
    },
    selsChange(sels) {
      this.sels = sels;
    },
    handleSearch() {
      this.page.pageIndex = 1;
      this.getWeChatKeyword();
    },
    handleSelectWeChat(val) {
      this.selectWeChat = val;
      this.page.pageIndex = 1;
      this.getWeChatKeyword();
    },
    handleCurrentChange(index) {
      this.page.pageIndex = index;
      this.getWeChatKeyword();
    },
    getWeChatKeyword() {
      this.listLoading = true;
      let para = {
        PageIndex: this.page.pageIndex,
        PageSize: this.page.pageSize,
        strOrderByFileds: '',
      };
      if (this.filters.name) {
        para.conditions = 'key contains ' + this.filters.name;
      }
      if (!this.selectWeChat) {
        this.$message.error("请选择要操作的公众号");
        return;
      }
      if (this.filters.name) {
        para.conditions += ' & publicAccount = ' + this.selectWeChat;
      }else{
        para.conditions = 'publicAccount = ' + this.selectWeChat;
      }

      getWeChatKeyword(para)
        .then(res => {
          this.listLoading = false;
          if (res.data.success) {
            this.tableData = res.data.response.data;
            this.total = res.data.response.dataCount
          } else {
            this.$message.error(res.data.msg);
          }
        });
    },
    handleDel() {
      if (!this.currentRow) {
        this.$message.error("请选择要操作的数据行");
        return;
      }
      this.$confirm("确认删除吗？", "提示", {}).then(() => {
        removeWeChatKeyword({ id: this.currentRow.Id }).then(res => {
          if (res.data.success) {
            this.getWeChatKeyword();
            this.$message.success("删除成功!");
          } else {
            this.$message.error(res.data.msg);
          }
        });
      });
    },
    handleEdit() {
      //编辑
      if (!this.currentRow) {
        this.$message.error("请选择要操作的数据行");
        return;
      }
      this.editFormVisible = true;
      this.editType = "edit";
      this.editForm = Object.assign({}, this.currentRow);

    },
    editFocusSubmit() {
      this.$refs.editForm.validate(valid => {
        if (valid) {
          this.$confirm("确认提交吗？", "提示", {}).then(() => {
            this.editLoading = true;
            if (this.editType == "add") {
              //console.log(this.editForm);
              //var postPara = this.editForm;
              addWeChatAccount(this.editForm)
                .then(res => {
                  this.editLoading = false;
                  if (res.data.success) {
                    this.getWeChatAccount();
                    this.editFormVisible = false;
                    this.$message.success("添加成功!");
                  } else {
                    this.$message.error(res.data.msg);
                  }
                })
            } else if (this.editType == "edit") {
              console.log(this.editForm);
              //var postPara = this.editForm;
              updateWeChatAccount(this.editForm)
                .then(res => {
                  this.editLoading = false;
                  if (res.data.success) {
                    this.getWeChatAccount();
                    this.editFormVisible = false;
                    this.$message.success("修改成功!");
                  } else {
                    this.$message.error(res.data.msg);
                  }

                })
            }
          });
        }
      });
    },
    handleAdd() {
      //新增
      this.editFormVisible = true;
      this.editType = "add";
      this.editForm = Object.assign({ publicAccount: this.selectWeChat, media_type: 'text' });

    },
    editSubmit() {
      //保存
      console.info("this.editForm", this.editForm)
      this.$refs.editForm.validate(valid => {
        if (valid) {
          this.$confirm("确认提交吗？", "提示", {}).then(() => {
            this.editLoading = true;
            if (this.editType == "add") {
              //console.log(this.editForm);
              //var postPara = this.editForm;
              addWeWeChatKeyword(this.editForm)
                .then(res => {
                  this.editLoading = false;
                  if (res.data.success) {
                    this.getWeChatKeyword();
                    this.editFormVisible = false;
                    this.$message.success("添加成功!");
                  } else {
                    this.$message.error(res.data.msg);
                  }

                });
            } else if (this.editType == "edit") {
              //console.log(this.editForm);
              //var postPara = this.editForm;
              updateWeChatKeyword(this.editForm)
                .then(res => {
                  this.editLoading = false;
                  if (res.data.success) {
                    this.getWeChatKeyword();
                    this.editFormVisible = false;
                    this.$message.success("修改成功!");
                  } else {
                    this.$message.error(res.data.msg);
                  }
                });
            }
          });
        }
      });
    },
    batchRemove() {
      if (!(this.sels.length > 0)) {
        this.$message.error("请选择要操作的数据行");
        return;
      }
      this.$confirm("确认批量删除吗？", "提示").then(() => {
        //批量删除
        var ids = this.sels.map(t => t.Id).join(",");
        batchDeleteWeChatKeyword({ ids: ids })
          .then(res => {
            if (res.data.success) {
              this.getWeChatKeyword();
              this.$message.success("批量成功!");
            } else {
              this.$message.error(res.data.msg);
            }
          });
      });
    },
  },
  mounted() {
  }
};
</script> 
