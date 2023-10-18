<template>
    <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true" :model="para" @submit.native.prevent>
                <el-form-item>
                    <el-input v-model="para.key" placeholder="标题/内容"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="searchWeChatAccount">查询</el-button>
                    <el-button type="primary" @click="handleAdd">新增</el-button>

                    <el-button type="primary" @click="handleEnable">开启所有</el-button>

                    <el-button type="primary" @click="handleDisable">禁用所有</el-button>
                </el-form-item>
                <!--<el-form-item>-->
                <!--<el-button type="primary" @click="handleAdd">新增</el-button>-->
                <!--</el-form-item>-->
            </el-form>
        </el-col>

        <!--列表-->
        <el-table :data="tableData" highlight-current-row v-loading="listLoading" @selection-change="selsChange"
            style="width: 100%;">
            <el-table-column type="index" width="80"></el-table-column>
            <el-table-column prop="title" label="标题/备注" width="150"></el-table-column>
            <el-table-column prop="content" show-overflow-tooltip label="内容" min-width="130"></el-table-column>
            <el-table-column prop="Enabled" label="状态" width="100">
                <template slot-scope="scope">
                    <el-tag :type="(scope.row.Enabled ? 'success' : 'info')"> {{ (scope.row.Enabled ? '启用' : '禁用')
                    }}</el-tag>
                </template>
            </el-table-column>


            <el-table-column label="操作" fixed="right" width="150">
                <template slot-scope="scope">
                    <el-button size="mini" type="primary" plain @click="handleEdit(scope.row)">编辑</el-button>
                    <el-button size="mini" type="danger" plain @click="handleDel(scope.row)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <!--工具条-->
        <el-col :span="24" class="toolbar">
            <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="page.pageSize"
                :total="page.pageTotal" style="float:right;"></el-pagination>
        </el-col>

        <!--编辑界面-->
        <el-dialog :title="editType" :visible.sync="editFormVisible" v-model="editFormVisible"
            :close-on-click-modal="false">
            <el-form :model="editForm" label-width="200px" :rules="editFormRules" ref="editForm" label-position="top">
                <el-form-item label="标题/备注" prop="title">
                    <el-input v-model="editForm.title" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="内容" prop="content">
                    <el-input type="textarea" v-model="editForm.content" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="状态" prop="content">
                    <el-switch style="display: block" v-model="editForm.Enabled" active-color="#13ce66"
                        inactive-color="#ff4949" active-text="启用" inactive-text="禁用">
                    </el-switch>
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
    getNsBanner,
    delNsBanner,
    addNsBanner,
    updateNsBanner,
    enableAllBanner,
    disableAllBanner
} from "../../api/api";
export default {
    name: "WeChatCompany",
    data() {
        return {
            para: {
                key: ""
            },
            listLoading: false,
            tableData: [],
            tableUser: [],
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
        handleEnable() {
            this.$confirm("确认一键开启所有吗？", "提示", {}).then(() => {
                enableAllBanner().then(res => {
                    if (res.data.success) {
                        this.searchWeChatAccount();
                        this.$message({
                            message: "更新成功!",
                            type: "success"
                        });
                    }
                });
            });
        },
        handleDisable() {
            this.$confirm("确认一键禁用所有吗？", "提示", {}).then(() => {
                disableAllBanner().then(res => {
                    if (res.data.success) {
                        this.searchWeChatAccount();
                        this.$message({
                            message: "更新成功!",
                            type: "success"
                        });
                    }
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
        searchWeChatAccount() {
            this.listLoading = true;
            getNsBanner()
                .then(res => {
                    this.listLoading = false;
                    if (res.data.success) {
                        this.tableData = res.data.response.data;
                        this.page.pageTotal = res.data.response.dataCount;
                    }
                });
        },
        handleDel(row) {
            this.$confirm("确认删除吗？", "提示", {}).then(() => {
                var servers = [];
                servers.push(row.Id);
                delNsBanner(servers).then(res => {
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
            this.editForm = Object.assign({ Enabled: true });

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
                            addNsBanner(this.editForm)
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
                            updateNsBanner(this.editForm)
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
                    var servers = this.sels.map(t => t.Id);
                    delNsBanner(servers)
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
  