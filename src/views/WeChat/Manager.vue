<template>
  <section>
    <!--工具条-->

    <toolbar :buttonList="buttonList" @callFunction="callFunction"></toolbar>

    <!--列表-->
    <el-table :data="tableData" highlight-current-row v-loading="listLoading" @selection-change="selsChange"
      @current-change="selectCurrentRow" style="width: 100%;">
      <el-table-column type="selection" width="60"></el-table-column>
      <el-table-column type="index" width="80"></el-table-column>
      <el-table-column prop="publicAccount" label="微信公众号ID" width sortable></el-table-column>
      <el-table-column prop="publicNick" label="微信公众名称" width sortable></el-table-column>

      <el-table-column prop="isFocusReply" label="是否关注回复" width sortable></el-table-column>
      <el-table-column prop="replyType" label="回复类型" width sortable></el-table-column>
      <el-table-column prop="replyText" label="回复文本" width sortable></el-table-column>
      <el-table-column prop="replyID" label="回复媒体ID" width sortable></el-table-column>
      <el-table-column prop="replyTitle" label="回复媒体标题" width sortable></el-table-column>
      <el-table-column prop="replyDescription" label="回复媒体描述" width sortable></el-table-column>
      <el-table-column prop="weChatNick" label="微信用户名称" width sortable></el-table-column>
      <el-table-column prop="tokenExpiration" label="token过期时间" width sortable></el-table-column>
      <el-table-column prop="Enabled" label="状态" width="" sortable>
        <template slot-scope="scope">
          <el-tag :type="scope.row.Enabled ? 'success' : 'danger'" disable-transitions>{{ scope.row.Enabled ? "正常" : "禁用"
          }}
          </el-tag>
        </template>
      </el-table-column>
    </el-table>
    <!--工具条-->
    <el-col :span="24" class="toolbar">
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :page-size="page.pageSize"
        :page-sizes="[10, 100, 1000]" layout="total, sizes, prev, pager, next" :total="page.pageTotal"
        style="float:right;">
      </el-pagination>
    </el-col>
    <!--编辑界面-->
    <el-dialog :title="editType" :visible.sync="editFormVisible" v-model="editFormVisible" :close-on-click-modal="false">
      <el-form :model="editForm" label-width="200px" :rules="editFormRules" ref="editForm">
        <el-form-item label="微信公众号ID" prop="publicAccount">
          <el-input v-model="editForm.publicAccount" auto-complete="off"
            :disabled="editType == 'edit' ? true : false"></el-input>
        </el-form-item>
        <el-form-item label="微信公众号名称" prop="publicNick">
          <el-input v-model="editForm.publicNick" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="微信ID" prop="weChatAccount">
          <el-input v-model="editForm.weChatAccount" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="微信名称" prop="weChatNick">
          <el-input v-model="editForm.weChatNick" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="AppID(应用ID)" prop="appid">
          <el-input v-model="editForm.appid" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="APPSecret(应用密钥)" prop="appsecret">
          <el-input v-model="editForm.appsecret" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="Token(交互-认证用)" prop="interactiveToken">
          <el-input v-model="editForm.interactiveToken" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="Token(令牌-推送用)" prop="token">
          <el-input v-model="editForm.token" auto-complete="off" :disabled="1 == 1 ? true : true"></el-input>
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input v-model="editForm.remark" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="状态" prop="Enabled">
          <el-select v-model="editForm.Enabled" placeholder="请选择状态">
            <el-option v-for="item in statusList" :key="item.value" :label="item.LinkUrl" :value="item.value"></el-option>

          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="editFormVisible = false">取消</el-button>
        <el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
      </div>
    </el-dialog>

    <!-- 编辑关注 -->
    <el-dialog :title="editType" :visible.sync="editFocusVisible" v-model="editFocusVisible"
      :close-on-click-modal="false">
      <el-form :model="editForm" label-width="200px" :rules="editFormRules" ref="editForm">
        <el-form-item label="微信公众号ID" prop="publicAccount">
          <el-input v-model="editForm.publicAccount" auto-complete="off" :disabled="true"></el-input>
        </el-form-item>
        <el-form-item label="微信公众号名称" prop="publicNick">
          <el-input :disabled="true" v-model="editForm.publicNick" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="是否关注回复" prop="isFocusReply">
          <el-radio v-model="editForm.isFocusReply" :label="true">是</el-radio>
          <el-radio v-model="editForm.isFocusReply" :label="false">否</el-radio>
        </el-form-item>
        <el-form-item v-if="editForm.isFocusReply" label="回复类型" prop="replyType">
          <el-select v-model="editForm.replyType" placeholder="请选择">
            <el-option label="文本" value="text"></el-option>
            <el-option label="图片" value="image"></el-option>
            <el-option label="语音" value="voice"></el-option>
            <el-option label="视频" value="video"></el-option>
            <el-option label="缩略图" value="thumb"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item v-if="editForm.replyType != 'text' && editForm.isFocusReply" label="回复媒体ID" prop="replyID">

          <el-row v-if="editForm.replyType == 'video'">
            <el-col>
              视频标题:<el-input style="width:calc(100% - 100px);margin-left: 5px;" v-model="editForm.replyTitle"
                auto-complete="off"></el-input>
            </el-col>
            <el-col style="margin-top: 5px;margin-bottom: 5px;">
              视频描述:<el-input style="width:calc(100% - 100px);margin-left: 5px;" v-model="editForm.replyDescription"
                auto-complete="off"></el-input>
            </el-col>
          </el-row>
          <el-row :gutter="10">
            <el-col style="width: calc(100% - 170px);">
              <el-input v-model="editForm.replyID" type="textarea" auto-complete="off" :disabled="true"></el-input>
            </el-col>
            <el-col style="width: 60px;">
              <el-upload class="upload-demo" action="#" :before-upload="beforeUpload">
                <el-link type="primary">新上传</el-link>
              </el-upload>
            </el-col>
            <el-col style="width: 80px;">
              <el-link type="primary" @click="handleMedia">选择</el-link>
            </el-col>
          </el-row>
        </el-form-item>
        <el-form-item v-if="editForm.replyType == 'text' && editForm.isFocusReply" label="回复文字" prop="replyText">
          <el-input type="textarea" :rows="5" v-model="editForm.replyText" auto-complete="off"></el-input>
        </el-form-item>

      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="editFocusVisible = false">取消</el-button>
        <el-button type="primary" @click.native="editSubmitFocus" :loading="editLoading">提交</el-button>
      </div>
    </el-dialog>

    <el-dialog title="素材选择" :visible.sync="visibleMedia" :close-on-click-modal="false" append-to-body>

      <el-table :data="mediaList" style="width: 100%" highlight-current-row @current-change="handleSelectMedia">
        <el-table-column prop="name" label="名称" min-width="100" show-overflow-tooltip>
        </el-table-column>
        <el-table-column prop="url" label="地址" width="100" show-overflow-tooltip>
        </el-table-column>
        <el-table-column prop="description" label="描述" width="100" show-overflow-tooltip>
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
  removeWeChatAccount,
  addWeChatAccount,
  updateWeChatAccount,
  batchDeleteChatAccount,
  refreshWeChatToken,
  UpdateWeChatFile,
  GetWeChatMediaList,
  updateWeChatAccountFocus
} from "../../api/api";
import { getButtonList } from "../../promissionRouter";
import Toolbar from "../../components/Toolbar";
export default {
  components: { Toolbar },
  data() {
    return {
      filters: {
        name: ""
      },
      buttonList: [],
      listLoading: false,
      tableData: [],
      sels: [],
      currentRow: null,
      page: {
        pageSize: 10,
        pageIndex: 1,
        pageTotal: 0
      },
      editFormVisible: false,
      editFocusVisible: false,
      editLoading: false,
      editType: "add", //默认新增类型
      editForm: {},
      statusList: [{ LinkUrl: '激活', value: true }, { LinkUrl: '禁用', value: false }],
      editFormRules: {
        publicAccount: [
          { required: true, message: "请输入微信公众号ID", trigger: "blur" }
        ],
        publicNick: [
          { required: true, message: "请输入微信公众号名称", trigger: "blur" }
        ],
        appid: [
          { required: true, message: "请输入AppID(应用ID)", trigger: "blur" }
        ],
        appsecret: [
          {
            required: true,
            message: "请输入APPSecret(应用密钥)",
            trigger: "blur"
          }
        ],
        interactiveToken: [
          {
            required: true,
            message: "请输入Token(交互-认证用)",
            trigger: "blur"
          }
        ],
        Enabled: [
          {
            required: true,
            message: "请选择状态",
            trigger: "blur"
          }
        ]
      },
      curMediaPage: 1,
      mediaTotal: 1,
      mediaList: [],
      visibleMedia: false,
      curSelectMedia: {},
    };
  },
  created() {
    this.getWeChatAccount();
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
      this.editForm.replyID = this.curSelectMedia.media_id
      if (this.editForm.replyType == 'video') {

        this.editForm.replyTitle = this.curSelectMedia.name
        this.editForm.replyDescription = this.curSelectMedia.description

      }
      this.visibleMedia = false
    },
    handleMedia() {
      this.curMediaPage = 1
      this.GetWeChatMediaList()
    },
    GetWeChatMediaList() {
      this.mediaList = []
      GetWeChatMediaList({ id: this.editForm.publicAccount, type: this.editForm.replyType, page: this.curMediaPage })
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
      if (!this.editForm.replyType) {
        this.$message.error("请选择媒体类型再上传");
        return false;
      }
      if (this.editForm.replyType == 'video') {
        if (!this.editForm.replyTitle || !this.editForm.replyDescription) {
          this.$message.error("请先填写视频的标题和描述");
          return false;
        }
      }
      // const isJPG = file.type === 'image/jpeg';
      // const isLt2M = file.size / 1024 / 1024 < 2;
      var formData = new FormData();
      formData.append("media", file);

      var des = { "title": this.editForm.replyTitle, "introduction": this.editForm.replyDescription }
      formData.append("description", JSON.stringify(des));
      UpdateWeChatFile(this.editForm.publicAccount, this.editForm.replyType, formData).then(res => {
        if (res.data.success) {
          this.$set(this.editForm, "replyID", res.data.response.media_id)
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
    handleCurrentChange(index) {
      this.page.pageIndex = index;
      this.getWeChatAccount();
    },
    handleSizeChange(val) {
      this.page.pageIndex = 1;
      this.page.pageSize = val;
      this.getWeChatAccount();
    },
    getWeChatAccount() {
      this.listLoading = true;
      let para = {
        PageIndex: this.page.pageIndex,
        PageSize: this.page.pageSize,
        strOrderByFileds: '',
      };
      if (this.filters.name) {
        para.conditions = 'publicNick like ' + this.filters.name + ' | ' + 'publicAccount like ' + this.filters.name;
      }
      getWeChatAccount(para)
        .then(res => {
          this.listLoading = false;
          if (res.data.success) {
            this.tableData = res.data.response.data;
            this.page.pageTotal = res.data.response.dataCount
          }
        });
    },
    handleRefreshWeChatToken() {
      if (!this.currentRow) {
        this.$message.error("请选择要操作的数据行");
        return;
      }
      refreshWeChatToken({ id: this.currentRow.publicAccount }).then(res => {
        if (res.data.success) {
          this.getWeChatAccount();
          this.$message.success("刷新Token成功!");
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
        removeWeChatAccount({ id: this.currentRow.publicAccount }).then(res => {
          if (res.data.success) {
            this.getWeChatAccount();
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

    handleEditFocus() {
      //编辑关注
      if (!this.currentRow) {
        this.$message.error("请选择要操作的数据行");
        return;
      }
      this.editFocusVisible = true;
      this.editType = "edit";
      this.editForm = Object.assign({}, this.currentRow);

    },
    editSubmitFocus() {
      //编辑
      this.$refs.editForm.validate(valid => {
        if (valid) {
          this.$confirm("确认提交吗？", "提示", {}).then(() => {
            this.editLoading = true;
            updateWeChatAccountFocus(this.editForm)
              .then(res => {
                this.editLoading = false;
                if (res.data.success) {
                  this.getWeChatAccount();
                  this.editFocusVisible = false;
                  this.$message.success("修改成功!");
                } else {
                  this.$message.error(res.data.msg);
                }

              })
          });
        }
      });
    },

    handleAdd() {
      //新增
      this.editFormVisible = true;
      this.editType = "add";
      this.editForm = Object.assign({});
    },
    editSubmit() {
      //保存
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


    batchRemove() {
      if (!(this.sels.length > 0)) {
        this.$message.error("请选择要操作的数据行");
        return;
      }
      this.$confirm("确认批量删除吗？", "提示").then(() => {
        //批量删除
        var ids = this.sels.map(t => t.publicAccount).join(",");
        batchDeleteChatAccount({ ids: ids })
          .then(res => {
            if (res.data.success) {
              this.getWeChatAccount();
              this.$message.success("批量删除成功!");
            } else {
              this.$message.error(res.data.msg);
            }
          })
      });
    },
    callFunction(item) {
      this.filters = {
        name: item.search
      };
      this[item.Func].apply(this, item);
    },
  },
  mounted() {
    let that = this;
    let routers = window.localStorage.router ? JSON.parse(window.localStorage.router) : [];
    this.buttonList = getButtonList(this.$route.path, routers);
  }
};
</script> 
