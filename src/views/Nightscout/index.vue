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
                    <el-button type="primary" @click="handleView(true)">预览</el-button>
                    <el-button type="primary" @click="handleView(false)">预览(备用)</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <!--列表-->
        <el-table :data="tableData" highlight-current-row @selection-change="selsChange"
            style="width: 100%;">
            <el-table-column type="selection" width="60"></el-table-column>
            <el-table-column type="index" width="80"></el-table-column>
            <el-table-column show-overflow-tooltip prop="name" label="名称" width="100"></el-table-column>
            <el-table-column show-overflow-tooltip prop="url" label="访问地址" width="250">
                <template slot-scope="scope">
                    {{ (scope.row.url ? 'https://' : '') }}{{ scope.row.url }}
                </template>
            </el-table-column>
            <el-table-column show-overflow-tooltip prop="startTime" label="开始时间" width="100">
                <template slot-scope="scope">
                    {{ (scope.row.startTime.replace(" 00:00:00", "")) }}
                </template>
            </el-table-column>
            <el-table-column show-overflow-tooltip prop="endTime" label="结束时间" width="100">
                <template slot-scope="scope">
                    {{ (scope.row.endTime.replace(" 00:00:00", "")) }}
                </template>
            </el-table-column>
            <el-table-column show-overflow-tooltip prop="passwd" label="密码" width="150"></el-table-column>
            <el-table-column show-overflow-tooltip prop="instanceIP" label="实例IP" width="120"></el-table-column>
            <el-table-column show-overflow-tooltip prop="serviceName" label="服务名称" width="200"></el-table-column>
            <el-table-column show-overflow-tooltip prop="isRefresh" label="强制刷新" width="90">
                <template slot-scope="scope">
                    <el-tag :type="scope.row.isRefresh ? 'warning' : ''">{{ scope.row.isRefresh ? '是' : '否' }}</el-tag>
                </template>
            </el-table-column>
            <el-table-column show-overflow-tooltip prop="isConnection" label="是否接入" width="90">
                <template slot-scope="scope">
                    <el-tag :type="scope.row.isConnection ? 'success' : ''">{{ scope.row.isConnection ? '是' : '否' }}</el-tag>
                </template>
            </el-table-column>


            <el-table-column show-overflow-tooltip prop="backupurl" label="备用访问" width="350">
                <template slot-scope="scope">
                    {{ (scope.row.backupurl ? 'https://' : '') }}{{ scope.row.backupurl }}
                </template>
            </el-table-column>

            <el-table-column show-overflow-tooltip prop="remark" label="备注" width="120"></el-table-column>
            <el-table-column show-overflow-tooltip prop="tel" label="电话" width="120"></el-table-column>

            <el-table-column label="操作" fixed="right" width="100">
                <template slot-scope="scope">
                    <el-button size="mini" type="primary" plain @click="handleEdit(scope.row)">编辑</el-button>
                    <el-dropdown trigger="click">
                        <span class="el-dropdown-link">
                            更多操作<i class="el-icon-arrow-down el-icon--right"></i>
                        </span>
                        <el-dropdown-menu slot="dropdown">
                            <el-dropdown-item icon="el-icon-plus"
                                @click.native="handleBind(scope.row)">获取绑定二维码</el-dropdown-item>
                            <el-dropdown-item icon="el-icon-close-notification"
                                @click.native="handleUnbind(scope.row)">解除绑定</el-dropdown-item>
                            <el-dropdown-item icon="el-icon-s-order"
                                @click.native="handleLog(scope.row)">操作日志</el-dropdown-item>

                            <el-dropdown-item icon="el-icon-s-order"
                                @click.native="handleRefresh(scope.row)">强制刷新</el-dropdown-item>
                            <el-dropdown-item icon="el-icon-delete"
                                @click.native="handleDel(scope.row)">删除</el-dropdown-item>
                        </el-dropdown-menu>
                    </el-dropdown>
                </template>
            </el-table-column>
        </el-table>
        <!--翻页-->
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="page.pageIndex"
            :page-sizes="[10, 100, 500, 1000]" :page-size="page.pageSize" layout="total, sizes, prev, pager, next, jumper"
            :total="page.pageTotal"></el-pagination>

        <!--编辑界面-->
        <el-dialog :title="editType" :visible.sync="editFormVisible" v-model="editFormVisible"
            :close-on-click-modal="false">
            <el-form :model="editForm" label-width="200px" :rules="editFormRules" ref="editForm">
                <el-form-item label="名称" prop="name">
                    <el-input v-model="editForm.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-tooltip class="item" effect="dark" content="不需要额外添加https否则会出问题" placement="top">
                    <el-form-item label="访问地址" prop="url">
                        <el-input v-model="editForm.url">
                            <template slot="prepend">https://</template>
                        </el-input>
                    </el-form-item>
                </el-tooltip>
                <el-form-item label="开始日期" prop="startTime">
                    <el-date-picker type="date" placeholder="选择日期" v-model="editForm.startTime" value-format="yyyy-MM-dd"
                        :picker-options="pickerOptions"></el-date-picker>
                </el-form-item>
                <el-form-item label="结束日期" prop="endTime">
                    <el-date-picker type="date" placeholder="选择日期" v-model="editForm.endTime" value-format="yyyy-MM-dd"
                        :picker-options="pickerOptions"></el-date-picker>
                </el-form-item>
                <el-form-item label="密码" prop="passwd">
                    <el-input v-model="editForm.passwd" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="电话" prop="tel">
                    <el-input v-model="editForm.tel" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="备注" prop="remark">
                    <el-input v-model="editForm.remark" auto-complete="off"></el-input>
                </el-form-item>
                <el-tooltip class="item" effect="dark" content="一般情况下不要乱动" placement="top">
                    <el-form-item label="实例IP" prop="instanceIP">

                        <el-input v-model="editForm.instanceIP" auto-complete="off"></el-input>

                    </el-form-item>
                </el-tooltip>
                <el-tooltip class="item" effect="dark" content="一般情况下不要乱动" placement="top">
                    <el-form-item label="服务名称" prop="serviceName">

                        <el-input v-model="editForm.serviceName" auto-complete="off"></el-input>

                    </el-form-item>
                </el-tooltip>
                <el-tooltip class="item" effect="dark" content="设置后每次编辑都会重启NS服务" placement="top">
                    <el-form-item label="强制刷新" prop="isRefresh">

                        <el-radio v-model="editForm.isRefresh" :label="true">是</el-radio>
                        <el-radio v-model="editForm.isRefresh" :label="false">否</el-radio>

                    </el-form-item>
                </el-tooltip>
                <el-tooltip class="item" effect="dark" content="设置后绑定微信后就可以推送了,需要强制刷新一次" placement="top">
                    <el-form-item label="是否接入" prop="isConnection">


                        <el-radio v-model="editForm.isConnection" :label="true">是</el-radio>
                        <el-radio v-model="editForm.isConnection" :label="false">否</el-radio>


                    </el-form-item>
                </el-tooltip>
                <el-tooltip class="item" effect="dark" content="不需要额外添加https否则会出问题" placement="top">
                    <el-form-item label="备用访问" prop="backupurl">
                        <el-input v-model="editForm.backupurl">
                            <template slot="prepend">https://</template>
                        </el-input>
                    </el-form-item>
                </el-tooltip>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="editFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
            </div>
        </el-dialog>

        <el-dialog title="动态血糖监测" :visible.sync="show" v-if="show" :fullscreen="true">

            <el-form :inline="true" label-position="right" label-width="100px">
                <el-form-item label="显示名称">
                    <el-checkbox v-model="showTitle"></el-checkbox>
                </el-form-item>
                <el-form-item label="排列方式">
                    <el-checkbox-group v-model="showCount">
                        <el-checkbox :true-label="1" :false-label="3">1排1个</el-checkbox>
                        <el-checkbox :true-label="2" :false-label="3">1排2个</el-checkbox>
                        <el-checkbox :true-label="3" :false-label="3">1排3个</el-checkbox>
                    </el-checkbox-group>
                </el-form-item>
            </el-form>
            <el-row :gutter="10">
                <el-col :span="getShowSpan()" :key="index" v-for="(item, index) in sels">
                    <el-link v-show="showTitle" icon="el-icon-s-custom">{{ item.name }}</el-link>
                    <iframe allowTransparency="true" frameborder="no" border="0" marginwidth="0" marginheight="0"
                        scrolling="no" height="300px" width="100%"
                        :src="(isNew ? ('https://' + item.url) : ('https://' + item.backupurl))"></iframe>
                </el-col>
            </el-row>
            <div slot="footer" class="dialog-footer">
                <el-button type="primary" @click.native="show = false">关闭</el-button>
            </div>
        </el-dialog>


        <el-dialog title="微信绑定二维码" :visible.sync="showBind" width="300px">
            <canvas style="height: 250px;width: 250px;" id="canvas"></canvas>
            <div slot="footer" class="dialog-footer">
                <el-button type="primary" @click.native="showBind = false">关闭</el-button>
            </div>
        </el-dialog>

        <el-dialog title="操作日志" :visible.sync="showLog">
            <el-table :data="tableLog" highlight-current-row style="width: 100%;">
                <el-table-column type="index" width="80"></el-table-column>
                <el-table-column show-overflow-tooltip prop="content" label="日志" min-width="350"></el-table-column>
                <el-table-column show-overflow-tooltip prop="success" label="状态" width="90">
                    <template slot-scope="scope">

                        <el-tag :type="(scope.row.success ? '' : 'danger')"> {{ (scope.row.success ? '成功' : '失败') }}
                        </el-tag>
                    </template>
                </el-table-column>
            </el-table>
            <div slot="footer" class="dialog-footer">
                <el-button type="primary" @click.native="showLog = false">关闭</el-button>
            </div>
        </el-dialog>

    </section>
</template>
  
<script>
import {
    getNightscout,
    delNightscout,
    addNightscout,
    updateNightscout,
    Refresh,
    GetWeChatCode,
    UnbindWeChat,
    GetLog
} from "../../api/api";
import QRCode from "qrcode";
export default {
    components: {},
    name: "Nightscout",
    data() {
        return {
            para: {

            },
            tableData: [],
            tableUser: [],
            tableLog: [],
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
                ],
                passwd: [
                    { required: true, message: "密码不能为空", trigger: "blur" }
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
            showTitle: true,
            showCount: 3,
            isNew: true,
            showBind: false,
            showLog: false
        };
    },
    created() {
        this.handleSearch();
    },
    methods: {
        handleRefresh(row) {
            Refresh({ id: row.Id }).then(res => {

                if (res.data && res.data.success) {
                    this.$message({
                        message: res.data.msg || "刷新成功!",
                        type: "error"
                    });
                } else {
                    this.$message({
                        message: res.data.msg || "刷新失败!",
                        type: "error"
                    });
                }
            })
        },
        handleBind(row) {

            GetWeChatCode({ id: row.Id }).then(res => {

                if (res.data && res.data.success && res.data.response.usersData.url) {
                    this.showBind = true
                    this.$nextTick(() => {
                        var canvas = document.getElementById("canvas");
                        QRCode.toCanvas(canvas, res.data.response.usersData.url, { width: 250, height: 250, }, function (error) {
                            if (error) console.error(error);
                        });
                    });
                } else {
                    this.$message({
                        message: res.data.msg || "获取失败!",
                        type: "error"
                    });
                }
            })

        },
        handleUnbind(row) {
            UnbindWeChat({ id: row.Id }).then(res => {
                if (res.data && res.data.success) {
                    this.$message({
                        message: "解绑成功!",
                        type: "success"
                    });
                } else {
                    this.$message({
                        message: "解绑失败!",
                        type: "error"
                    });
                }
            })
        },
        handleLog(row) {
            this.tableLog = []
            GetLog({ id: row.Id }).then(res => {
                if (res.data && res.data.success) {
                    this.showLog = true
                    this.tableLog = res.data.response.data
                } else {
                    this.$message({
                        message: "获取失败!",
                        type: "error"
                    });
                }
            })

        },
        getShowSpan() {
            if (this.showCount == 1) return 24
            if (this.showCount == 2) return 12
            if (this.showCount == 3) return 8
            return 24
        },
        handleView(isNew) {
            if (!this.sels.length) {
                this.$message({
                    message: "请选择要预览的数据!",
                    type: "error"
                });
            } else {
                this.isNew = isNew
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
        handleSizeChange(size) {
            this.page.pageIndex = 1;
            this.page.pageSize = size;
            this.handleSearch();
        },
        handleSearch() {
            getNightscout({ page: this.page.pageIndex, key: this.para.name, pageSize: this.page.pageSize })
                .then(res => {
                    if (res.data.success) {
                        this.tableData = res.data.response.data;
                        this.page.pageTotal = res.data.response.dataCount
                    }
                });
        },
        handleDel(row) {
            this.$confirm("确认删除吗[" + row.name + "]？", "提示", {}).then(() => {
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
            this.editType = "编辑";
            this.editForm = Object.assign({}, row);

        },
        handleAdd() {
            //新增
            this.editFormVisible = true;
            this.editType = "添加";
            this.editForm = Object.assign({});

        },
        editSubmit() {
            //保存
            this.$refs.editForm.validate(valid => {
                if (valid) {
                    this.$confirm("确认提交吗？", "提示", {}).then(() => {
                        this.editLoading = true;
                        if (this.editType == "添加") {
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
                        } else if (this.editType == "编辑") {
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
<style scoped>
.el-dropdown-link {
    cursor: pointer;
    color: #409EFF;
}

.el-icon-arrow-down {
    font-size: 12px;
}

.demonstration {
    display: block;
    color: #8492a6;
    font-size: 14px;
    margin-bottom: 20px;
}
</style>
  