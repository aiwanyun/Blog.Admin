<template>
  <section>
    <!--工具条-->
    <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
      <el-form :inline="true" :model="para" @submit.native.prevent>
        <el-form-item>
          <el-input v-model="para.publicAccount" placeholder="标题/内容"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="searchWeChatAccount">查询</el-button>
          <el-button type="primary" @click="handleAdd">新增</el-button> 
        </el-form-item>
        <!--<el-form-item>-->
        <!--<el-button type="primary" @click="handleAdd">新增</el-button>-->
        <!--</el-form-item>-->
      </el-form>
    </el-col>

    <!--列表-->
    <el-table
      :data="tableData"
      highlight-current-row
       
      v-loading="listLoading"
      @selection-change="selsChange"
      style="width: 100%;"
    >
      <el-table-column type="index" width="80"></el-table-column>
      <el-table-column prop="serverName" label="服务器名称" width="" ></el-table-column>
      <el-table-column prop="serverIp" label="服务器地址" width ></el-table-column>
      <el-table-column prop="serverPort" label="服务器端口" width ></el-table-column>
      <el-table-column prop="curInstanceIp" label="当前实例IP" width sortable></el-table-column> 
      <el-table-column prop="curInstanceIpSerial" label="当前实例IP序列" width sortable></el-table-column> 
      <el-table-column prop="instanceIpTemplate" label="实例ip模板" width sortable></el-table-column> 
      <el-table-column prop="curExposedPort" label="当前暴露端口" width sortable></el-table-column> 
      <el-table-column prop="curServiceSerial" label="当前服务序列" width sortable></el-table-column> 
      <el-table-column prop="mongoIp" label="数据库地址" width sortable></el-table-column> 
      <el-table-column prop="mongoPort" label="数据库端口" width sortable></el-table-column> 
      <el-table-column prop="mongoLoginName" label="数据库账号" width sortable></el-table-column> 
      <el-table-column prop="mongoLoginPassword" label="数据库密码" width sortable></el-table-column> 
      <el-table-column prop="remark" label="备注" width sortable></el-table-column> 
      
      
      <el-table-column label="操作"  fixed="right">
        <template slot-scope="scope">
          <el-button size="mini" type="primary" plain @click="handleEdit(scope.row)">编辑</el-button>
          <el-button size="mini" type="danger" plain @click="handleDel(scope.row)">删除</el-button> 
        </template>
      </el-table-column>
    </el-table>
    <!--工具条-->
    <el-col :span="24" class="toolbar">
      <el-pagination
        layout="prev, pager, next"
        @current-change="handleCurrentChange"
        :page-size="page.pageSize"
        :total="page.pageTotal"
        style="float:right;"
      ></el-pagination>
    </el-col>

    <!--编辑界面-->
    <el-dialog
      :title="editType"
      :visible.sync="editFormVisible"
      v-model="editFormVisible"
      :close-on-click-modal="false"
    >
      <el-form :model="editForm" label-width="200px" :rules="editFormRules" ref="editForm">
        <el-form-item label="服务器名称" prop="serverName">
          <el-input v-model="editForm.serverName" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="服务器地址" prop="serverIp">
          <el-input v-model="editForm.serverIp" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="服务器登录账号" prop="serverLoginName">
          <el-input v-model="editForm.serverLoginName" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="服务器登录密码" prop="serverLoginPassword">
          <el-input v-model="editForm.serverLoginPassword" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="服务器端口" prop="serverPort">
          <el-input v-model="editForm.serverPort" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="当前实例IP" prop="curInstanceIp">
          <el-input v-model="editForm.curInstanceIp" auto-complete="off"></el-input>
        </el-form-item>
        <el-form-item label="当前实例IP序列" prop="curInstanceIpSerial">
          <el-input v-model="editForm.curInstanceIpSerial" auto-complete="off"></el-input>
        </el-form-item> 
        <el-form-item label="实例ip模板" prop="instanceIpTemplate">
          <el-input v-model="editForm.instanceIpTemplate" auto-complete="off"></el-input>
        </el-form-item> 
        <el-form-item label="当前暴露端口" prop="curExposedPort">
          <el-input v-model="editForm.curExposedPort" auto-complete="off"></el-input>
        </el-form-item> 
        <el-form-item label="当前服务序列" prop="curServiceSerial">
          <el-input v-model="editForm.curServiceSerial" auto-complete="off"></el-input>
        </el-form-item> 
        <el-form-item label="数据库地址" prop="mongoHost">
          <el-input v-model="editForm.mongoIp" auto-complete="off"></el-input>
        </el-form-item> 
        <el-form-item label="数据库端口" prop="mongoPort">
          <el-input v-model="editForm.mongoPort" auto-complete="off"></el-input>
        </el-form-item> 
        <el-form-item label="数据库账号" prop="mongoLoginName">
          <el-input v-model="editForm.mongoLoginName" auto-complete="off"></el-input>
        </el-form-item> 
        <el-form-item label="数据库密码" prop="mongoLoginPassword">
          <el-input v-model="editForm.mongoLoginPassword" auto-complete="off"></el-input>
        </el-form-item> 
        <el-form-item label="备注" prop="remark">
          <el-input v-model="editForm.remark" auto-complete="off"></el-input>
        </el-form-item> 
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="editFormVisible = false">取消</el-button>
        <el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
      </div>
    </el-dialog>
  </section>
</template>

<script>
import {
  getNsServer, 
  delNsServer,
  addNsServer,
  updateNsServer,
} from "../../api/api";
export default {
  name: "WeChatCompany",
  data() {
    return {
      para: {
        CompanyID: "test"
      },
      listLoading: false,
      tableData: [],
      tableUser:[],
      sels: [],
      page: {
        pageSize: 50,
        pageIndex: 1,
        pageTotal: 50
      },
      editFormVisible: false,
      editLoading: false,
      editType: "add", //默认新增类型
      editForm: {},
      editFormRules: {
        serveraddress: [
          { required: true, message: "服务器地址不能为空", trigger: "blur" }
        ],
        serverport: [
          { required: true, message: "服务器端口不能为空", trigger: "blur" }
        ]
      }
    };
  },
  created() {
    this.searchWeChatAccount();
  },
  methods: {
    selsChange(sels) {
      this.sels = sels;
    },
    handleCurrentChange(index) {
      this.page.pageIndex = index;
      this.searchWeChatAccount();
    },
    searchWeChatAccount() {
      this.listLoading = true;
      getNsServer()
        .then(res => {
          this.listLoading = false;
          if(res.data.success){
            this.tableData = res.data.response.data;
            this.page.pageTotal =res.data.response.dataCount;
          }
        });
    },   
    handleDel(row) {
      this.$confirm("确认删除吗？", "提示", {}).then(() => {
        var servers = [];
        servers.push(row.id);
        delNsServer(servers).then(res => {
          if (res.data.success) {
            this.searchWeChatAccount();
            this.$message({
              message: "删除成功!",
              type: "success"
            });
          }
        });
      });
    },
    handleEdit(row) {
      //编辑
      this.editFormVisible = true;
      this.editType = "edit";
      this.editForm = Object.assign({}, row);
       
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
              addNsServer(this.editForm)
                .then(res => {
                  this.editLoading = false;
                  if (res.data.success) {
                    this.$message({
                       message: "添加成功!",
                      type: "success"
                    });
                    this.searchWeChatAccount();
                    this.editFormVisible = false;
                  }
                  
                });
            } else if (this.editType == "edit") {
              //console.log(this.editForm);
              //var postPara = this.editForm;
              updateNsServer(this.editForm)
                .then(res => {
                  this.editLoading = false;
                  if (res.data.success) {
                    this.$message({
                        message: "编辑成功!",
                      type: "success"
                    });
                    this.searchWeChatAccount();
                    this.editFormVisible = false;
                  } 
                });
            }
          });
        }
      });
    },
    batchRemove() {
      this.$confirm("确认批量删除吗？", "提示").then(() => {
        //批量删除
        if (this.sels.length > 0) {
          var servers = this.sels.map(t => t.id);
          delNsServer(servers)
            .then(res => {
              if (res.data.success) {
                this.$message({
                    message: "批量删除成功!",
                  type: "success"
                });
                this.searchWeChatAccount();
              }
            });
        } else {
          this.$message({
            message: "请先选择要删除的数据!",
            type: "info"
          });
        }
      });
    }
  },
  mounted() {
    let that = this;
  }
};
</script> 
