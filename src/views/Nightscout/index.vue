<template>
    <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true" :model="para" @submit.native.prevent>
                <el-form-item>
                    <el-input v-model="para.name" placeholder="标题/内容"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleSearch">查询</el-button>
                    <el-button type="primary" @click="handleAdd">新增</el-button>
                    <el-button type="primary" @click="handleView">预览</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <!--列表-->
        <el-table :data="tableData" highlight-current-row v-loading="listLoading" @selection-change="selsChange"
            style="width: 100%;">
            <el-table-column type="selection" width="60"></el-table-column>
            <el-table-column type="index" width="80"></el-table-column>
            <el-table-column prop="name" label="名称" width sortable></el-table-column>
            <el-table-column prop="url" label="访问地址" width sortable></el-table-column>
            <el-table-column prop="tel" label="电话" width sortable></el-table-column>
            <el-table-column prop="startTime" label="开始时间" width sortable></el-table-column>
            <el-table-column prop="endTime" label="结束时间" width sortable></el-table-column>
            <el-table-column prop="remark" label="备注" width sortable></el-table-column>
            <el-table-column prop="backupurl" label="备用访问" width sortable></el-table-column>
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
            <el-form :model="editForm" label-width="200px" :rules="editFormRules" ref="editForm">
                <el-form-item label="名称" prop="name">
                    <el-input v-model="editForm.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="访问地址" prop="url">
                    <el-input v-model="editForm.url" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="电话" prop="tel">
                    <el-input v-model="editForm.tel" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="开始日期" prop="startTime">
                    <el-date-picker type="date" placeholder="选择日期" v-model="editForm.startTime" value-format="yyyy-MM-dd"
                        :picker-options="pickerOptions"></el-date-picker>
                </el-form-item>
                <el-form-item label="结束日期" prop="endTime">
                    <el-date-picker type="date" placeholder="选择日期" v-model="editForm.endTime" value-format="yyyy-MM-dd"
                        :picker-options="pickerOptions"></el-date-picker>
                </el-form-item>
                <el-form-item label="备注" prop="remark">
                    <el-input v-model="editForm.remark" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="备用访问" prop="backupurl">
                    <el-input v-model="editForm.backupurl" auto-complete="off"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="editFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
            </div>
        </el-dialog>

        <el-dialog title="动态血糖监测" :visible.sync="show" v-model="editFormVisible" :fullscreen="true">
        
            <el-row style="height: calc(100vh - 200px);">
                <el-col :span="8" :key="index" v-for="(item, index) in sels"> <iframe height="300px" width="100%"
                        :src="item.url"></iframe></el-col>
            </el-row>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="show = false">关闭</el-button>
            </div>
        </el-dialog>
    </section>
</template>
  
<script>
import {
    getNightscout,
    delNightscout,
    addNightscout,
    updateNightscout
} from "../../api/api";
export default {
    name: "Nightscout",
    data() {
        return {
            para: {

            },
            listLoading: false,
            tableData: [],
            tableUser: [],
            sels: [],
            page: {
                pageSize: 10,
                pageIndex: 1,
                pageTotal: 0
            },
            editFormVisible: false,
            editLoading: false,
            editType: "add", //默认新增类型
            editForm: {},
            editFormRules: {
                name: [
                    { required: true, message: "名称不能为空", trigger: "blur" }
                ],
                url: [
                    { required: true, message: "访问地址不能为空", trigger: "blur" }
                ],
                startTime: [
                    { required: true, message: "开始日期不能为空", trigger: "blur" }
                ],
                endTime: [
                    { required: true, message: "结束日期不能为空", trigger: "blur" }
                ]
            },
            pickerOptions: {
                shortcuts: [
                    {
                        text: "今天",
                        onClick(picker) {
                            picker.$emit("pick", new Date());
                        },
                    },
                    {
                        text: "明天",
                        onClick(picker) {
                            const date = new Date();
                            date.setTime(date.getTime() + 3600 * 1000 * 24);
                            picker.$emit("pick", date);
                        },
                    },
                    {
                        text: "一周后",
                        onClick(picker) {
                            const date = new Date();
                            date.setTime(date.getTime() + 3600 * 1000 * 24 * 7);
                            picker.$emit("pick", date);
                        },
                    },
                    {
                        text: "一月后(30)",
                        onClick(picker) {
                            const date = new Date();
                            date.setTime(date.getTime() + 3600 * 1000 * 24 * 30);
                            picker.$emit("pick", date);
                        },
                    },
                    {
                        text: "一年后(365)",
                        onClick(picker) {
                            const date = new Date();
                            date.setTime(date.getTime() + 3600 * 1000 * 24 * 365);
                            picker.$emit("pick", date);
                        },
                    },
                ],
            },
            show: false,
        };
    },
    created() {
        this.handleSearch();
    },
    methods: {
        handleView() {
            if (!this.sels.length) {
                this.$message({
                    message: "请选择要预览的数据!",
                    type: "error"
                });
            } else {
                this.show = true
            }
        },
        selsChange(sels) {
            this.sels = sels;
        },
        handleCurrentChange(index) {
            this.page.pageIndex = index;
            this.handleSearch();
        },
        handleSearch() {
            this.listLoading = true;
            getNightscout({ page: this.page.pageIndex, key: this.para.name, pageSize: this.page.pageSize })
                .then(res => {
                    this.listLoading = false;
                    if (res.data.success) {
                        this.tableData = res.data.response.data;
                        this.page.pageTotal = res.data.response.dataCount
                    }
                });
        },
        handleDel(row) {
            this.$confirm("确认删除吗？", "提示", {}).then(() => {
                delNightscout({ id: row.Id }).then(res => {
                    if (res.data.success) {
                        this.handleSearch();
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
                            addNightscout(this.editForm)
                                .then(res => {
                                    this.editLoading = false;
                                    if (res.data.success) {
                                        this.$message({
                                            message: "添加成功!",
                                            type: "success"
                                        });
                                        this.handleSearch();
                                        this.editFormVisible = false;
                                    }

                                });
                        } else if (this.editType == "edit") {
                            //console.log(this.editForm);
                            //var postPara = this.editForm;
                            updateNightscout(this.editForm)
                                .then(res => {
                                    this.editLoading = false;
                                    if (res.data.success) {
                                        this.$message({
                                            message: "编辑成功!",
                                            type: "success"
                                        });
                                        this.handleSearch();
                                        this.editFormVisible = false;
                                    }
                                });
                        }
                    });
                }
            });
        },
    },
    mounted() {
        let that = this;
    }
};
</script> 
  