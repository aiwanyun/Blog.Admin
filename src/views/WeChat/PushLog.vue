<template>
  <section>
    <!--工具条-->
    <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
      <el-form :inline="true" @submit.native.prevent>
        <el-form-item>
          <el-select v-model="selectWeChat" placeholder="请选择要查询的公众号">
            <el-option v-for="item in wechats" :key="item.value" :label="item.label" :value="item.value">
              <span style="float: left">{{ item.label }}</span>
              <span style="float: right; color: #8492a6; font-size: 13px">{{ item.value }}</span>
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-select v-model="selectCompany" placeholder="请选择要查询的客户">
            <el-option v-for="item in companys" :key="item.value" :label="item.label" :value="item.value">
              <span style="float: left">{{ item.label }}</span>
              <span style="float: right; color: #8492a6; font-size: 13px">{{ item.value }}</span>
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-input clearable v-model="selectUser" placeholder="请选择要查询的用户"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" :disabled="selectWeChat == ''" @click="searchWeChatAccount">查询</el-button>
        </el-form-item>
      </el-form>
    </el-col>

    <!--列表-->
    <el-table :data="tableData" highlight-current-row v-loading="listLoading" @selection-change="selsChange"
      style="width: 100%;">
      <el-table-column type="index" width="80"></el-table-column>
      <el-table-column prop="PushLogPublicAccount" label="微信公众号" width="100"></el-table-column>
      <el-table-column prop="PushLogCompanyID" label="客户" width></el-table-column>
      <el-table-column prop="PushLogToUserID" label="员工ID" width></el-table-column>
      <el-table-column prop="PushLogOpenid" label="微信ID" width="300"></el-table-column>
      <el-table-column prop="PushLogTime" label="推送时间" width="250"></el-table-column>
      <el-table-column prop="PushLogStatus" label="推送状态" width></el-table-column>
      <el-table-column prop="PushLogRemark" label="信息" width></el-table-column>
      <el-table-column prop="PushLogIP" label="推送IP" width></el-table-column>
      <el-table-column prop="PushLogTemplateID" label="推送模板ID" width="150"></el-table-column>
      <el-table-column prop="PushLogContent" label="推送内容" width="500"></el-table-column>
    </el-table>
    <!--工具条-->
    <div class="block">
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="page.pageIndex"
        :hide-on-single-page="true" :page-sizes="[10, 100, 500, 1000]" :page-size="page.pageSize"
        layout="total, sizes, prev, pager, next, jumper" :total="page.pageTotal">
      </el-pagination>
    </div>
  </section>
</template>

<script>
import { getWeChatPushLog, getWeChatAccount, getWeChatCompany } from "../../api/api";
export default {
  name: "WeChatCompany",
  data() {
    return {
      wechats: [], //微信公众号列表
      companys: [], //客户列表
      selectWeChat: "", //当前选中的微信公众号 
      selectCompany: "", //当前选中的微信客户
      selectUser: '',//查询的用户
      listLoading: false,
      tableData: [],
      sels: [],
      page: {
        pageSize: 10,
        pageIndex: 1,
        pageTotal: 0
      }
    };
  },
  created() {
    this.getWeChats();
    this.getWeCompanys();
  },
  methods: {
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
    searchWeChatAccount() {
      this.listLoading = true;
      var pars = {
        PageIndex: this.page.pageIndex,
        PageSize: this.page.pageSize,
        strOrderByFileds: "Id desc",
        conditions: "PushLogPublicAccount = " + this.selectWeChat
      }
      if (this.selectCompany) {
        pars.conditions += " & PushLogCompanyID = " + this.selectCompany
      }
      if (this.selectUser) {
        pars.conditions += " & PushLogToUserID = " + this.selectUser
      }
      getWeChatPushLog(pars).then(res => {
        this.listLoading = false;
        console.log(res);
        if (res.data.success) {
          this.tableData = res.data.response.data;
          this.page.pageTotal = res.data.response.dataCount;
          this.$message({
            type: "success",
            message: "获取成功!"
          });
        }
      });
    },
    getWeCompanys() {
      getWeChatCompany().then(res => {
        this.companys = [];
        console.log(res);
        res.data.response.data.forEach(element => {
          this.companys.push({
            value: element.CompanyID,
            label: element.CompanyName
          });
        });
      });
    },
    getWeChats() {
      getWeChatAccount().then(res => {
        this.wechats = [];
        res.data.response.data.forEach(element => {
          this.wechats.push({
            value: element.publicAccount,
            label: element.publicNick
          });
        });
      });
    }
  },
  mounted() {
    let that = this;
  },
  watch: {
    selectWeChat: function (newName, oldName) {
      this.searchWeChatAccount();
    },
    selectCompany: function (newName, oldName) {
      this.searchWeChatAccount();
    }
  }
};
</script> 
