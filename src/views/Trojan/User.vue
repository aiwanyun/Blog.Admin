<template>
  <section>
    <!--工具条-->


    <!--工具条-->
    <el-row>
      <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
        <el-form :inline="true" @submit.native.prevent>
          <el-form-item>
            <el-input v-model="filters.name" placeholder="标题/内容"></el-input>
            <el-checkbox v-model="filters.isuse" @change="searchMy">流量使用</el-checkbox>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="searchMy">查询</el-button> 
            <el-button type="primary" @click="addUserShow">添加</el-button>
          </el-form-item>
        </el-form>
      </el-col>
    </el-row>
    <!--列表-->
    <el-table :data="tableData" highlight-current-row v-loading="listLoading" @selection-change="selsChange"
      style="width: 100%;" show-summary :summary-method="getSummaries">
      <el-table-column type="selection" width="100"></el-table-column>
      <el-table-column type="index" width="100"></el-table-column>
      <el-table-column prop="username" min-label="用户名" width="150"></el-table-column>
      <el-table-column prop="quota" label="流量" min-width="150">
        <template slot-scope="scope">
          <el-tag>{{ scope.row.quota < 0 ? "无限流量" : getFlow(scope.row.quota) }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="upload" label="本月上传" min-width="150">
        <template slot-scope="scope">
          <el-tag>{{ getFlow(scope.row.upload) }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="download" label="本月下载" min-width="150">
        <template slot-scope="scope">
          <el-tag>{{ getFlow(scope.row.download) }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="total" label="本月累计" min-width="150">
        <template slot-scope="scope">
          <el-tag>{{ getFlow(scope.row.download + scope.row.upload) }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="历史统计" min-width="150">
        <template slot-scope="scope">
          <el-popover trigger="hover" placement="left">

            <template>


              <el-row>
                <el-col>用户名: <el-link type="primary"> {{ scope.row.username }} </el-link></el-col>
                <el-col>起止时间:<el-link type="success">{{ (scope.row.useList.length > 0 ?
                  scope.row.useList[scope.row.useList.length - 1].moth : '')
                }}-</el-link><el-link type="success">{{ (scope.row.useList.length > 1 ?
  scope.row.useList[0].moth : '')
                    }}</el-link></el-col>
              </el-row>
              <div style="height: 50vh;">
                <el-table height="100%" :data="scope.row.useList" show-summary :summary-method="getSummaries">
                  <el-table-column prop="moth" label="月度" width="80">
                  </el-table-column>
                  <el-table-column prop="up" label="上传" width="120">
                    <template slot-scope="scope">
                      <el-tag>{{ getFlow(scope.row.up) }}</el-tag>
                    </template>
                  </el-table-column>
                  <el-table-column prop="down" label="下载" width="120">
                    <template slot-scope="scope">
                      <el-tag>{{ getFlow(scope.row.down) }}</el-tag>
                    </template>
                  </el-table-column>
                  <el-table-column prop="total" label="累计" width="120">
                    <template slot-scope="scope">
                      <el-tag>{{ getFlow(scope.row.total) }}</el-tag>
                    </template>
                  </el-table-column>

                </el-table>
              </div>
            </template>
            <div slot="reference" class="name-wrapper">
              <el-tag size="medium">点击查看</el-tag>
            </div>
          </el-popover>
        </template>
      </el-table-column>
      <el-table-column prop="passwordshow" label="操作" width="150" fixed="right">
        <template slot-scope="scope">
          <el-dropdown>
            <span class="el-dropdown-link">
              操作
              <i class="el-icon-arrow-down el-icon--right"></i>
            </span>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item @click.native="resetFlow(scope.row)">重置流量</el-dropdown-item>
              <el-dropdown-item @click.native="limitFlowShow(scope.row)">限制流量</el-dropdown-item>
              <el-dropdown-item @click.native="resetPass(scope.row)">重置密码</el-dropdown-item>
              <el-dropdown-item @click.native="handleShow(scope.row)">获取链接</el-dropdown-item>
              <el-dropdown-item @click.native="editUserShow(scope.row)">编辑</el-dropdown-item>
              <el-dropdown-item @click.native="delUser(scope.row)">删除</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </template>
      </el-table-column>

    </el-table>
    <!--翻页-->
    <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="page.pageIndex"
      :page-sizes="[10, 100, 500, 1000]" :page-size="page.pageSize" layout="total, sizes, prev, pager, next, jumper"
      :total="page.pageTotal"></el-pagination>
    <!-- 二维码 -->
    <!-- 服务器 -->
    <el-dialog title="服务器列表" :visible.sync="serverFormVisible" v-model="serverFormVisible" :close-on-click-modal="false"
      width="500px">
      <div style="text-align: center;">
        <el-form :inline="false" @submit.native.prevent>
          <el-form-item label="Clash订阅选择">
            <el-link type="success" v-clipboard:copy="clashApi" v-clipboard:success="onCopy" v-clipboard:error="onError"
              style="float:left">点击复制-{{ showPass }}</el-link>
            <el-link type="primary" v-clipboard:copy="clashApiBackup" v-clipboard:success="onCopy"
              v-clipboard:error="onError" style="float:left">备用</el-link>
          </el-form-item>
          <el-form-item label="普通订阅选择">
            <el-link type="success" v-clipboard:copy="normalApi" v-clipboard:success="onCopy"
              v-clipboard:error="onError" style="float:left">点击复制-{{ showPass }}</el-link>
          </el-form-item>
          <el-form-item label="单节点选择">
            <el-form-item>
              <el-select v-model="selectServerId" placeholder="请选择要链接的服务器" filterable style="width:300px;float:left">
                <el-option v-for="item in servers" :key="item.name" :label="item.name" :value="item.value">
                  <span style="float: left">{{ item.name }}</span>
                </el-option>
              </el-select>
            </el-form-item>
          </el-form-item>

          <div v-if="showCode">
            <el-form-item>
              <canvas id="canvas"></canvas>
              <br />
              <el-link type="success" v-clipboard:copy="showValue" v-clipboard:success="onCopy"
                v-clipboard:error="onError">{{ showValue }}</el-link>
              <br />
            </el-form-item>
          </div>
        </el-form>
      </div>
    </el-dialog>
    <!-- 添加用户 -->
    <el-dialog :title="addType" :visible.sync="addFormVisible" v-model="addFormVisible" :close-on-click-modal="false"
      width="650px">
      <el-form :inline="true" @submit.native.prevent>
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addUserData.username" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="流量" prop="quota">
          <el-tooltip class="item" effect="dark" content="-1为无限流量" placement="top-start">
            <el-input-number :disabled="addType == 'edit'" v-model="addUserData.quota" :min="-1" :max="99999"
              auto-complete="off"></el-input-number>
          </el-tooltip>
          <el-select :disabled="addType == 'edit'" v-model="optionsValue" placeholder="请选择" style="width:73px;">
            <el-option v-for="item in options" :key="item.txt" :label="item.txt" :value="item.txt"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="addFormVisible = false">取消</el-button>
        <el-button type="primary" @click.native="submitUser" :loading="addLoading">提交</el-button>
      </div>
    </el-dialog>
    <el-dialog title="流量限制" :visible.sync="limitFormVisible" v-model="limitFormVisible" :close-on-click-modal="false"
      width="500px">
      <el-form :inline="true" @submit.native.prevent>
        <el-form-item label="流量" prop="quota">
          <el-tooltip class="item" effect="dark" content="-1为无限流量" placement="top-start">
            <el-input-number v-model="limitData.quota" :min="-1" :max="99999" auto-complete="off"></el-input-number>
          </el-tooltip>

          <el-select v-model="optionsValue" placeholder="请选择" style="width:73px;">
            <el-option v-for="item in options" :key="item.txt" :label="item.txt" :value="item.txt"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="limitFormVisible = false">取消</el-button>
        <el-button type="primary" @click.native="limitFlow" :loading="limiyLoading">提交</el-button>
      </div>
    </el-dialog>
  </section>
</template>

<script>
import {
  getTrojanUser,
  addTrojanUser,
  updateTrojanUser,
  delTrojanUser,
  resetTrojanFlow,
  limitTrojanFlow,
  resetTrojanPass,
  getSpliceServer
} from "../../api/api";

import QRCode from "qrcode";
import { getButtonList } from "../../promissionRouter";
export default {
  data() {
    return {
      filters: {
        name: "",
        isuse: true
      },
      buttonList: [],
      listLoading: false,
      tableData: [],
      sels: [],
      page: {
        pageSize: 10,
        pageIndex: 1,
        pageTotal: 0
      },
      addFormVisible: false,
      limitFormVisible: false,
      serverFormVisible: false,
      showValue: "",
      normalApi: "",
      clashApi: "",
      clashApiBackup: "",
      showPass: "",
      addLoading: false,
      addType: "", //编辑/新增
      limiyLoading: false,
      addUserDataTemplate: {
        username: "",
        quota: 0
      },
      addUserData: {},
      options: [{ txt: "KB" }, { txt: "MB" }, { txt: "GB" }],
      optionsValue: "GB",
      limitData: {
        id: 0,
        quota: 0
      },
      servers: [],
      selectServer: {},
      selectServerId: null,
      selectRow: {},
      showCode: false,
    };
  },
  created() {
    this.searchWeChatAccount();
  },
  methods: {
    getSummaries(param) {
      const { columns, data } = param;
      const sums = [];
      columns.forEach((column, index) => {
        if (index === 0) {
          sums[index] = '合计';
          return;
        }
        const values = data.map(item => Number(item[column.property]));
        if ((column.property === 'upload'
          || column.property === 'download' || column.property === 'up'
          || column.property === 'down' || column.property === 'total')) {
          sums[index] = values.reduce((prev, curr) => {
            const value = Number(curr);
            if (!isNaN(value)) {
              return prev + curr;
            } else {
              return prev;
            }
          }, 0);
          sums[index] = this.getFlow(sums[index]);
        } else {
          sums[index] = '';
        }
      });

      return sums;
    },
    getFlow(quota) {
      if (quota / 1024 / 1024 / 1024 >= 1) {
        return (quota / 1024 / 1024 / 1024).toFixed(3) + "GB";
      }
      if (quota / 1024 / 1024 >= 1) {
        return (quota / 1024 / 1024).toFixed(3) + "MB";
      }
      return (quota / 1024).toFixed(3) + "KB";
    },
    callFunction(item) {
      this.filters.name = item.search
      this[item.Func].apply(this, item);
    },
    onCopy(e) {
      // 复制成功
      this.$message({
        message: "复制成功！",
        type: "success"
      });
    },
    onError(e) {
      // 复制失败
      this.$message({
        message: "复制失败！",
        type: "error"
      });
    },
    resetPass(row) {
      this.$confirm("确定要重置密码么?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          var users = [];
          users.push(row.id);
          resetTrojanPass(users).then(res => {
            if (res.data.success) {
              this.searchWeChatAccount();
            }
          });
        })
        .catch(() => { });
    },
    limitFlowShow(row) {
      this.limitFormVisible = true;
      this.limitData.id = row.id;
      if (row.quota / 1024 / 1024 / 1024 >= 1) {
        this.limitData.quota = row.quota / 1024 / 1024 / 1024;
        this.optionsValue = "GB";
      } else if (row.quota / 1024 / 1024 >= 1) {
        this.limitData.quota = row.quota / 1024 / 1024;
        this.optionsValue = "MB";
      } else if (row.quota / 1024 >= 1) {
        this.limitData.quota = row.quota / 1024;
        this.optionsValue = "KB";
      } else {
        this.limitData.quota = row.quota;
      }
    },
    limitFlow() {
      this.limiyLoading = true;
      var users = [this.limitData.id];
      var quota = 0;
      if (this.limitData.quota <= 0) {
        quota = this.limitData.quota;
      } else if (this.optionsValue == "GB") {
        quota = this.limitData.quota * 1024 * 1024 * 1024;
      } else if (this.optionsValue == "MB") {
        quota = this.limitData.quota * 1024 * 1024;
      } else if (this.optionsValue == "KB") {
        quota = this.limitData.quota * 1024;
      }
      limitTrojanFlow({ users: users, quota: quota })
        .then(res => {
          if (res.data.success) {
            this.limitFormVisible = false;
            this.searchWeChatAccount();
          }
          this.limiyLoading = false;
        })
        .catch(err => {
          console.error(err);
          this.limiyLoading = false;
        });
    },
    resetFlow(row) {
      this.$confirm("确定要重置流量么?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          var users = [];
          users.push(row.id);
          resetTrojanFlow(users).then(res => {
            if (res.data.success) {
              this.searchWeChatAccount();
            }
          });
        })
        .catch(() => { });
    },
    delUser(row) {
      this.$confirm("确定要删除么?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          var users = [];
          users.push(row.id);
          delTrojanUser(users).then(res => {
            if (res.data.success) {
              this.searchWeChatAccount();
            }
          });
        })
        .catch(() => { });
    },
    addUserShow() {
      this.addType = "add";
      this.addFormVisible = true;
      this.addLoading = false;
      this.addUserData = Object.assign({}, this.addUserDataTemplate);
    },
    editUserShow(row) {
      this.addType = "edit";
      this.addFormVisible = true;
      this.addLoading = false;
      this.addUserData = Object.assign({}, row);
      if (this.addUserData.quota != -1 && this.addUserData.quota != 0) {
        //流量转换
        var realFlow = getFlow(this.addUserData.quota);
        var kb = realFlow.replace(/\d+\.\d+/, "");
        var num = realFlow.replace(kb, "");
        this.addUserData.quota = num;
        this.optionsValue = kb;
      }
    },
    submitUser() {
      if (this.addType == "edit") {
        this.updateUser();
      } else {
        this.addUser();
      }
    },
    updateUser() {
      var send = Object.assign({}, this.addUserData);
      updateTrojanUser(send)
        .then(res => {
          if (res.data.success) {
            this.searchWeChatAccount();
            this.addFormVisible = false;
          }
          this.addLoading = false;
        })
        .catch(err => {
          console.error(err);
          this.addLoading = false;
        });
    },
    addUser() {
      this.addLoading = true;
      var send = Object.assign({}, this.addUserData);
      if (send.quota > 0) {
        if (this.optionsValue == "KB") send.quota = send.quota * 1024;
        if (this.optionsValue == "MB") send.quota = send.quota * 1024 * 1024;
        if (this.optionsValue == "GB")
          send.quota = send.quota * 1024 * 1024 * 1024;
      }
      addTrojanUser(send)
        .then(res => {
          if (res.data.success) {
            this.searchWeChatAccount();
            this.addFormVisible = false;
          }
          this.addLoading = false;
        })
        .catch(err => {
          console.error(err);
          this.addLoading = false;
        });
    },
    handleShow(row) {
      getSpliceServer({ id: row.passwordshow }).then(res => {
        if (res.data.success) {
          this.showPass = row.passwordshow;
          this.normalApi = res.data.response.normalApi + row.passwordshow;
          this.clashApi = res.data.response.clashApi + this.normalApi;
          this.clashApiBackup = res.data.response.clashApiBackup + this.normalApi;
          this.servers = res.data.response.list;
          this.serverFormVisible = true;
          this.selectRow = row;
          this.selectServerId = null;
          this.selectServer = {};
          if (this.showCode) this.showCode = false;
        }
      });
    },
    getQR(mylink) {
      this.showValue = mylink;
      this.$nextTick(() => {
        var canvas = document.getElementById("canvas");
        QRCode.toCanvas(canvas, mylink, function (error) {
          if (error) console.error(error);
        });
      });
    },
    selsChange(sels) {
      this.sels = sels;
    },
    handleCurrentChange(index) {
      this.page.pageIndex = index;
      this.searchWeChatAccount();
    },
    handleSizeChange(size) {
      this.page.pageIndex = 1;
      this.page.pageSize = size;
      this.searchWeChatAccount();
    },
    searchMy() {
      this.page.pageIndex = 1;
      this.searchWeChatAccount();
    },
    searchWeChatAccount() {
      this.listLoading = true;
      getTrojanUser({
        PageIndex: this.page.pageIndex,
        PageSize: this.page.pageSize,
        strOrderByFileds: "",
        isuse: this.filters.isuse,
        name: this.filters.name
      }).then(res => {
        this.listLoading = false;
        if (res.data.success) {
          for (let i = 0; i < res.data.response.data.length; i++) {
            let tempRow = res.data.response.data[i]
            tempRow.total = tempRow.upload + tempRow.download
            console.log(tempRow.total)
          }
          this.tableData = res.data.response.data;
          this.page.pageTotal = res.data.response.dataCount;
          this.$message({
            type: "success",
            message: "获取成功!"
          });
        }
      });
    }
  },
  mounted() {
    let that = this;
    let routers = window.localStorage.router
      ? JSON.parse(window.localStorage.router)
      : [];
    this.buttonList = getButtonList(this.$route.path, routers);
  },
  watch: {
    selectServerId: function (newValue, oldValue) {
      if (newValue) {
        this.showCode = true;
        this.getQR(newValue);
      }
    }
  }
};
</script> 
