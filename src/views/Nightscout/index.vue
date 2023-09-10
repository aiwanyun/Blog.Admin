<template>
    <section>
        <!--工具条-->
        <el-row>
            <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
                <el-form :inline="true" :model="para" @submit.native.prevent style="margin-top: 10px;">
                    <el-form-item>
                        <el-input clearable v-model="para.name" placeholder="标题/内容"
                            @keyup.enter.native.prevent="handleCurrentChange(1)"></el-input>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="handleCurrentChange(1)">查询</el-button>
                        <el-button type="primary" @click="handleAdd">新增</el-button>
                        <el-button type="primary" @click="handleView(true)">预览</el-button>
                        <el-button type="primary" @click="handleView(false)">预览-备用</el-button>
                    </el-form-item>
                    <el-form-item>
                        <label>状态:</label>
                        <el-badge :style="{ 'margin-left': index === 0 ? '0px' : '30px' }" :key="item.name"
                            v-for="(item, index) in summary.status" :value="item.count" class="item">
                            <el-tag @click="handleTag(item.name)" style="cursor:pointer;width: 60px;text-align: center;">{{
                                (item.name ? item.name : '未确认') }}</el-tag>
                        </el-badge>
                        <label style="margin-left: 10px;">来源:</label>
                        <el-badge :style="{ 'margin-left': index === 0 ? '0px' : '30px' }" :key="item.name"
                            v-for="(item, index) in summary.resource" :value="item.count" class="item">
                            <el-tag @click="handleTag(item.name)" style="cursor:pointer;width: 60px;text-align: center;;"
                                type="info">{{ (item.name ? item.name : '未确认')
                                }}</el-tag>
                        </el-badge>
                    </el-form-item>
                </el-form>
            </el-col>
        </el-row>
        <!--列表-->
        <el-table :data="tableData" highlight-current-row @selection-change="selsChange">
            <el-table-column type="selection" width="50"></el-table-column>
            <el-table-column type="index" width="40"></el-table-column>
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
            <el-table-column show-overflow-tooltip prop="instanceIP" label="实例IP" width="150"></el-table-column>
            <el-table-column show-overflow-tooltip prop="serviceName" label="服务名称" width="200"></el-table-column>
            <el-table-column show-overflow-tooltip prop="serverId" label="服务器" width="150">
                <template slot-scope="scope">{{
                    getServerName(scope.row)
                }}</template>
            </el-table-column>

            <el-table-column show-overflow-tooltip prop="serverId" label="服务网络" width="150">
                <template slot-scope="scope">{{
                    getCDNName(scope.row)
                }}</template>
            </el-table-column>
            <el-table-column show-overflow-tooltip prop="isRefresh" label="自动重启" width="90">
                <template slot-scope="scope">
                    <el-tag :type="scope.row.isRefresh ? 'warning' : ''">{{ scope.row.isRefresh ? '是' : '否' }}</el-tag>
                </template>
            </el-table-column>
            <el-table-column show-overflow-tooltip prop="isConnection" label="高低报警" width="90">
                <template slot-scope="scope">
                    <el-tag :type="scope.row.isConnection ? 'success' : ''">{{ scope.row.isConnection ? '是' : '否'
                    }}</el-tag>
                </template>
            </el-table-column>
            <el-table-column show-overflow-tooltip prop="isKeepPush" label="持续推送" width="90">
                <template slot-scope="scope">
                    <el-tag :type="scope.row.isKeepPush ? 'success' : ''">{{ scope.row.isKeepPush ? '是' : '否'
                    }}</el-tag>
                </template>
            </el-table-column>
            <el-table-column show-overflow-tooltip prop="isBindWeChat" label="绑定公众号" width="90">
                <template slot-scope="scope">
                    <el-tag :type="scope.row.isBindWeChat ? 'success' : ''">{{ scope.row.isBindWeChat ? '是' : '否'
                    }}</el-tag>
                </template>
            </el-table-column>
            <el-table-column show-overflow-tooltip prop="isBindMini" label="绑定小程序" width="90">
                <template slot-scope="scope">
                    <el-tag :type="scope.row.isBindMini ? 'success' : ''">{{ scope.row.isBindMini ? '是' : '否'
                    }}</el-tag>
                </template>
            </el-table-column>


            <el-table-column show-overflow-tooltip prop="status" label="状态" width="90"></el-table-column>
            <el-table-column show-overflow-tooltip prop="resource" label="来源" width="90"></el-table-column>
            <el-table-column show-overflow-tooltip prop="money" label="费用/元" width="90"></el-table-column>


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
                                @click.native="handleBind(scope.row)">获取公众号绑定二维码</el-dropdown-item>
                            <el-dropdown-item icon="el-icon-close-notification"
                                @click.native="handleUnbind(scope.row)">解除公众号绑定</el-dropdown-item>
                            <el-dropdown-item icon="el-icon-plus"
                                @click.native="handleBindMini(scope.row)">获取小程序绑定二维码</el-dropdown-item>
                            <el-dropdown-item icon="el-icon-close-notification"
                                @click.native="handleUnbindMini(scope.row)">解除小程序绑定</el-dropdown-item>
                            <el-dropdown-item icon="el-icon-refresh"
                                @click.native="handleRefresh(scope.row)">重启实例</el-dropdown-item>
                            <el-dropdown-item icon="el-icon-error"
                                @click.native="handleStop(scope.row)">停止实例</el-dropdown-item>
                            <el-dropdown-item icon="el-icon-s-open"
                                @click.native="handleReset(scope.row)">重置数据</el-dropdown-item>
                            <el-dropdown-item icon="el-icon-s-order"
                                @click.native="handleLog(scope.row)">操作日志</el-dropdown-item>
                            <el-dropdown-item icon="el-icon-delete"
                                @click.native="handleDel(scope.row)">删除</el-dropdown-item>
                            <el-dropdown-item icon="el-icon-document-copy"
                                @click.native="copy('https://' + scope.row.url)">复制url地址</el-dropdown-item>
                            <el-dropdown-item icon="el-icon-document-copy"
                                @click.native="copy(scope.row.passwd)">复制密码</el-dropdown-item>
                            <el-dropdown-item icon="el-icon-document-copy"
                                @click.native="copy('https://' + scope.row.passwd + '@' + scope.row.url + '/api/v1')">复制api地址</el-dropdown-item>
                            <el-dropdown-item icon="el-icon-document-copy"
                                @click.native="copy('https://' + scope.row.url + '/api/v1/entries')">复制entries地址</el-dropdown-item>
                            <el-dropdown-item icon="el-icon-document-copy"
                                @click.native="copy('https://' + scope.row.url + ' ' + scope.row.passwd + '\n' + 'https://' + scope.row.passwd + '@' + scope.row.url + '/api/v1')">复制url+passwd+api</el-dropdown-item>
                        </el-dropdown-menu>
                    </el-dropdown>
                </template>
            </el-table-column>
        </el-table>
        <!--翻页-->
        <el-pagination small layout="prev, pager, next" :limit.sync="page.pageSize" :total="page.pageTotal"
            :current-page.sync="page.pageIndex" @current-change="handleCurrentChange">
        </el-pagination>
        <!-- <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="page.pageIndex"
            :page-sizes="[10, 100, 500, 1000]" :page-size="page.pageSize" layout="total, sizes, prev, pager, next, jumper"
            :total="page.pageTotal"></el-pagination> -->

        <!--编辑界面-->
        <el-dialog :title="editType" :visible.sync="editFormVisible" v-model="editFormVisible"
            :close-on-click-modal="false">
            <el-form :model="editForm" label-width="200px" label-position="top" :rules="editFormRules" ref="editForm">
                <el-form-item label="名称" prop="name">
                    <el-input v-model="editForm.name" placeholder="如不填写,则自动生成" auto-complete="off"></el-input>
                </el-form-item>
                <el-tooltip class="item" effect="dark" content="不需要额外添加https否则会出问题" placement="top">
                    <el-form-item label="访问地址" prop="url">
                        <el-input v-model="editForm.url" placeholder="如不填写,则自动生成">
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
                    <el-input v-model="editForm.passwd" placeholder="如不填写,则自动生成" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="电话" prop="tel">
                    <el-input v-model="editForm.tel" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="备注" prop="remark">
                    <el-input v-model="editForm.remark" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="部署服务器" prop="serverId">
                    <el-select v-model="editForm.serverId" placeholder="请选择">
                        <el-option v-for="item in nsServer" :key="item.Id" :label="item.serverName + '(' + item.count + ')'"
                            :value="item.Id">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="服务网络" prop="cdn">
                    <el-select v-model="editForm.cdn" placeholder="请选择">
                        <el-option v-for="item in cdnList" :key="item.value" :label="item.name" :value="item.value">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-tooltip class="item" effect="dark" content="一般情况下不要乱动" placement="top">
                    <el-form-item label="实例IP" prop="instanceIP">
                        <el-input :disabled="true" v-model="editForm.instanceIP" auto-complete="off"
                            placeholder="自动生成"></el-input>
                    </el-form-item>
                </el-tooltip>
                <el-tooltip class="item" effect="dark" content="一般情况下不要乱动" placement="top">
                    <el-form-item label="服务名称" prop="serviceName">
                        <el-input :disabled="true" v-model="editForm.serviceName" auto-complete="off"
                            placeholder="自动生成"></el-input>
                    </el-form-item>
                </el-tooltip>
                <el-tooltip class="item" effect="dark" content="设置后每次编辑都会重启NS服务" placement="top">
                    <el-form-item label="自动重启" prop="isRefresh">
                        <el-radio v-model="editForm.isRefresh" :label="true">是</el-radio>
                        <el-radio v-model="editForm.isRefresh" :label="false">否</el-radio>
                    </el-form-item>
                </el-tooltip>
                <el-tooltip class="item" effect="dark" content="设置后绑定微信后就可以推送了,需要重启实例一次" placement="top">
                    <el-form-item label="高低报警" prop="isConnection">
                        <el-radio v-model="editForm.isConnection" :label="true">是</el-radio>
                        <el-radio v-model="editForm.isConnection" :label="false">否</el-radio>
                    </el-form-item>
                </el-tooltip>
                <el-tooltip class="item" effect="dark" content="设置持续推送后,需要重启实例一次" placement="top">
                    <el-form-item label="持续推送" prop="isKeepPush">
                        <el-radio v-model="editForm.isKeepPush" :label="true">是</el-radio>
                        <el-radio v-model="editForm.isKeepPush" :label="false">否</el-radio>
                    </el-form-item>
                </el-tooltip>

                <el-tooltip class="item" effect="dark" content="不需要额外添加https否则会出问题" placement="top">
                    <el-form-item label="备用访问" prop="backupurl">
                        <el-input v-model="editForm.backupurl">
                            <template slot="prepend">https://</template>
                        </el-input>
                    </el-form-item>
                </el-tooltip>
                <el-form-item label="状态" prop="status">
                    <el-select v-model="editForm.status" placeholder="请选择状态">
                        <el-option label="未启用" value="未启用"></el-option>
                        <el-option label="试用中" value="试用中"></el-option>
                        <el-option label="已付费" value="已付费"></el-option>
                        <el-option label="已到期" value="已到期"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="来源" prop="resource">
                    <el-select clearable v-model="editForm.resource" placeholder="请选择来源">
                        <el-option label="自来" value="自来"></el-option>
                        <el-option label="推广" value="推广"></el-option>
                        <el-option label="介绍" value="介绍"></el-option>
                        <el-option label="分成" value="分成"></el-option>
                        <el-option label="未确认" value="未确认"></el-option>
                    </el-select>
                </el-form-item>

                <el-form-item label="费用/元" prop="money">
                    <el-select v-model.number="editForm.money" filterable allow-create placeholder="请选择金额">
                        <el-option label="0" value="0"></el-option>
                        <el-option label="100" value="100"></el-option>
                        <el-option label="120" value="120"></el-option>
                        <el-option label="130" value="130"></el-option>
                        <el-option label="150" value="150"></el-option>
                        <el-option label="180" value="180"></el-option>
                        <el-option label="200" value="200"></el-option>
                    </el-select>
                </el-form-item>


                <el-form-item label="启用组件" prop="plugins_arr">
                    <el-select filterable style="width: 100%;" clearable multiple v-model="editForm.plugins_arr"
                        placeholder="请选择要启用的组件">
                        <el-option :key="item.key" v-for="item in plugins" :label="item.name + '-' + item.key"
                            :value="item.key"></el-option>
                    </el-select>
                </el-form-item>

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

        <el-dialog title="小程序绑定二维码" v-if="showMini" :visible.sync="showMini" width="300px">
            <el-image style="width: 250px; height: 250px" :src="'data:image/png;base64,' + this.curMini">
            </el-image>
            <div slot="footer" class="dialog-footer">
                <el-button type="primary" @click.native="showMini = false">关闭</el-button>
            </div>
        </el-dialog>

        <el-dialog title="操作日志" :visible.sync="showLog">
            <el-table :data="tableLog" highlight-current-row style="width: 100%;">
                <el-table-column type="index" width="80"></el-table-column>
                <el-table-column show-overflow-tooltip prop="CreateTime" label="创建时间" width="160"></el-table-column>
                <el-table-column show-overflow-tooltip prop="content" label="日志" min-width="350"></el-table-column>
                <el-table-column show-overflow-tooltip prop="success" label="状态" width="90">
                    <template slot-scope="scope">
                        <el-tag :type="(scope.row.success ? '' : 'danger')"> {{ (scope.row.success ? '成功' : '失败') }}
                        </el-tag>
                    </template>
                </el-table-column>
            </el-table>
            <el-pagination @size-change="handleSizeChangeLog" @current-change="handleCurrentChangeLog"
                :current-page="pageLog.pageIndex" :page-sizes="[10, 100, 500, 1000]" :page-size="pageLog.pageSize"
                layout="total, sizes, prev, pager, next, jumper" :total="pageLog.pageTotal"></el-pagination>
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
    GetWeChatMiniCode,
    UnbindWeChat,
    UnbindWeChatMini,
    GetLog,
    Reset,
    Stop,
    GetSummary,
    GetPlugins,
    getAllNsServer
} from "../../api/api";
import QRCode from "qrcode";
import util from "../../../util/date";
export default {
    components: {},
    name: "Nightscout",
    data() {
        return {
            para: {
                name: ''
            },
            tableData: [],
            tableUser: [],
            tableLog: [],
            summary: {
                status: [],
                resource: []
            },
            sels: [],
            page: {
                pageSize: 10,
                pageIndex: 1,
                pageTotal: 0
            },
            pageLog: {
                pageSize: 10,
                pageIndex: 1,
                pageTotal: 0
            },
            editFormVisible: false,
            editLoading: false,
            editType: "add", //默认新增类型
            editForm: {},
            editFormRules: {
                // name: [
                //     { required: true, message: "名称不能为空", trigger: "blur" }
                // ],
                // url: [
                //     { required: true, message: "访问地址不能为空", trigger: "blur" }
                // ],
                startTime: [
                    { required: true, message: "开始日期不能为空", trigger: "blur" }
                ],
                endTime: [
                    { required: true, message: "结束日期不能为空", trigger: "blur" }
                ],
                // passwd: [
                //     { required: true, message: "密码不能为空", trigger: "blur" }
                // ],
                status: [
                    { required: true, message: "状态不能为空", trigger: "blur" }
                ],
                plugins_arr: [
                    { required: true, message: "至少要选择一个组件", trigger: "blur" }
                ],
                serverId: [
                    { required: true, message: "部署服务器不能为空", trigger: "blur" }
                ],
                cdn: [
                    { required: true, message: "服务网络不能为空", trigger: "blur" }
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
            showMini: false,
            curMini: {},
            showLog: false,
            curRow: {},
            plugins: [],
            nsServer: [],
            cdnList: [
                {
                    name: "亚马逊云(国外)",
                    value: "aws"
                },
                {
                    name: "七牛云(国内-主用)",
                    value: "qiniu"
                },
                {
                    name: "阿里云(国内-aps)",
                    value: "aliyun"
                }, {
                    name: "腾讯云(国内-备用)",
                    value: "qcloud"
                }
            ]
        };
    },
    created() {
        this.handleSummary();
        this.handleSearch();
        this.GetPlugins();
        this.getAllNsServer();
    },
    methods: {
        copy(data) {
            let elInput = document.createElement('textarea')
            elInput.value = data
            document.body.appendChild(elInput)
            // 选择对象
            elInput.select()
            console.log(elInput, elInput.value)
            // 执行浏览器复制命令
            document.execCommand("Copy")
            elInput.remove()
            this.$message({
                message: "复制成功！",
                type: "success"
            });
        },
        getCDNName(row) {
            let findRow = this.cdnList.find(t => t.value === row.cdn)
            let tag = "";
            if (findRow) {
                tag = findRow.name
            }
            return tag;
        },
        getServerName(row) {
            let findRow = this.nsServer.find(t => t.Id === row.serverId)
            let tag = "";
            if (findRow) {
                tag += findRow.serverName
                if (row.exposedPort > 0) {
                    tag += "(" + row.exposedPort + ")";
                }
            }
            return tag;
        },
        getAllNsServer() {
            getAllNsServer().then(res => {
                if (res.data.success) {
                    this.nsServer = res.data.response
                }
            })
        },
        GetPlugins() {
            GetPlugins().then(res => {
                if (res.data.success) {
                    this.plugins = res.data.response
                } else {
                    this.$message({
                        message: res.data.msg || "插件获取失败!",
                        type: "error"
                    });
                }
            })
        },
        handleReset(row) {
            this.$confirm("确定重置[" + row.name + "]的数据吗？", "提示", {}).then(() => {
                Reset({ id: row.Id }).then(res => {

                    if (res.data && res.data.success) {
                        this.$message({
                            message: res.data.msg || "重置成功!",
                            type: "success"
                        });
                    } else {
                        this.$message({
                            message: res.data.msg || "重置失败!",
                            type: "error"
                        });
                    }
                })
            });
        },
        handleRefresh(row) {
            this.$confirm("确认刷新[" + row.name + "]的NS服务？", "提示", {}).then(() => {
                Refresh({ id: row.Id }).then(res => {

                    if (res.data && res.data.success) {
                        this.$message({
                            message: res.data.msg || "刷新成功!",
                            type: "success"
                        });
                    } else {
                        this.$message({
                            message: res.data.msg || "刷新失败!",
                            type: "error"
                        });
                    }
                })
            });

        },
        handleStop(row) {
            this.$confirm("确认停止[" + row.name + "]的NS服务？", "提示", {}).then(() => {
                Stop({ id: row.Id }).then(res => {
                    if (res.data && res.data.success) {
                        this.$message({
                            message: res.data.msg || "停止成功!",
                            type: "success"
                        });
                    } else {
                        this.$message({
                            message: res.data.msg || "停止失败!",
                            type: "error"
                        });
                    }
                })
            });

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
        handleBindMini(row) {

            GetWeChatMiniCode({ nsid: row.Id }).then(res => {

                if (res.data && res.data.success) {
                    this.curMini = res.data.response;
                    this.showMini = true

                } else {
                    this.$message({
                        message: res.data.msg || "获取失败!",
                        type: "error"
                    });
                }
            })




        },
        handleUnbind(row) {
            this.$confirm("确定解除[" + row.name + "]的公众号绑定吗？", "提示", {}).then(() => {
                UnbindWeChat({ id: row.Id }).then(res => {
                    if (res.data && res.data.success) {
                        this.$message({
                            message: res.data.msg || "解绑成功!",
                            type: "success"
                        });
                    } else {
                        this.$message({
                            message: res.data.msg || "解绑失败!",
                            type: "error"
                        });
                    }
                    this.handleSearch();
                })
            });

        },
        handleUnbindMini(row) {
            this.$confirm("确定解除[" + row.name + "]的小程序绑定吗？", "提示", {}).then(() => {
                UnbindWeChatMini({ nsid: row.Id }).then(res => {
                    if (res.data && res.data.success) {
                        this.$message({
                            message: res.data.msg || "解绑成功!",
                            type: "success"
                        });
                    } else {
                        this.$message({
                            message: res.data.msg || "解绑失败!",
                            type: "error"
                        });
                    }
                    this.handleSearch();
                })
            });
        },
        handleLog(row) {
            if (row) this.curRow = row
            this.tableLog = []
            GetLog({ id: this.curRow.Id, pageSize: this.pageLog.pageSize, page: this.pageLog.pageIndex }).then(res => {
                if (res.data && res.data.success) {
                    this.showLog = true
                    this.tableLog = res.data.response.data
                    this.pageLog.pageTotal = res.data.response.dataCount
                } else {
                    this.$message({
                        message: res.data.msg || "获取失败!",
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
        handleCurrentChangeLog(index) {
            this.pageLog.pageIndex = index;
            this.handleLog();
        },
        handleSizeChange(size) {
            this.page.pageIndex = 1;
            this.page.pageSize = size;
            this.handleSearch();
        },
        handleTag(key) {
            this.para.name = key;
            this.page.pageIndex = 1;
            this.handleSearch();
        },
        handleSizeChangeLog(size) {
            this.pageLog.pageIndex = 1;
            this.pageLog.pageSize = size;
            this.handleLog();
        },
        handleSearch() {
            getNightscout({ key: this.para.name, pageSize: this.page.pageSize, page: this.page.pageIndex })
                .then(res => {
                    if (res.data.success) {
                        this.tableData = res.data.response.data;
                        this.page.pageTotal = res.data.response.dataCount
                    } else {
                        this.tableData = []
                        this.$message({
                            message: res.data.msg || "获取失败!",
                            type: "error"
                        });
                    }
                });

        },
        handleSummary() {
            this.summary.status = []
            this.summary.resource = []
            GetSummary().then(res => {
                if (res.data.success) {
                    this.summary.status = res.data.response.status
                    this.summary.resource = res.data.response.resource
                } else {
                    this.$message({
                        message: res.data.msg || "统计数据获取失败!",
                        type: "error"
                    });
                }

            })
        },
        handleDel(row) {
            this.$confirm("确认删除[" + row.name + "]的NS服务吗？", "提示", {}).then(() => {
                delNightscout({ id: row.Id }).then(res => {
                    if (res.data.success) {
                        this.handleSearch();
                        this.$message({
                            message: res.data.msg || "删除成功!",
                            type: "success"
                        });
                    } else {
                        this.$message({
                            message: res.data.msg || "删除失败!",
                            type: "error"
                        });
                    }
                });
            });
        },
        handleEdit(row) {
            //编辑
            this.getAllNsServer();
            this.editType = "编辑";
            this.editForm = Object.assign({}, row);
            if (row.plugins) {
                this.$set(this.editForm, "plugins_arr", JSON.parse(row.plugins))
            } else {
                this.$set(this.editForm, "plugins_arr", JSON.parse(JSON.stringify(this.plugins.map(t => t.key))))
            }
            this.editFormVisible = true;

        },
        handleAdd() {
            //新增
            this.getAllNsServer();
            this.editType = "添加";
            let date = new Date();

            let startDate = util.formatDate.format(date, "yyyy-MM-dd");
            date.setTime(date.getTime() + 3600 * 1000 * 24 * 365);
            let endDate = util.formatDate.format(date, "yyyy-MM-dd");
            this.editForm = Object.assign({ money: 0, startTime: startDate, endTime: endDate, isRefresh: false, isConnection: true, isKeepPush: false, status: '未启用', resource: '未确认' });
            this.$set(this.editForm, "plugins_arr", JSON.parse(JSON.stringify(this.plugins.map(t => t.key))))
            this.editFormVisible = true;
        },
        editSubmit() {
            //保存
            this.$refs.editForm.validate(valid => {
                if (valid) {
                    this.$confirm("确认提交吗？", "提示", {}).then(() => {
                        this.editLoading = true;
                        this.editForm.plugins = JSON.stringify(this.editForm.plugins_arr)
                        if (this.editType == "添加") {
                            addNightscout(this.editForm)
                                .then(res => {
                                    this.editLoading = false;
                                    if (res.data.success) {
                                        this.$message({
                                            message: res.data.msg || "添加成功!",
                                            type: "success"
                                        });
                                        this.handleSearch();
                                        this.editFormVisible = false;
                                        this.handleSummary();
                                    } else {
                                        this.$message({
                                            message: res.data.msg || "添加失败!",
                                            type: "error"
                                        });
                                    }

                                });
                        } else if (this.editType == "编辑") {
                            updateNightscout(this.editForm)
                                .then(res => {
                                    this.editLoading = false;
                                    if (res.data.success) {
                                        this.$message({
                                            message: res.data.msg || "编辑成功!",
                                            type: "success"
                                        });
                                        this.handleSearch();
                                        this.editFormVisible = false;
                                        this.handleSummary();
                                    } else {
                                        this.$message({
                                            message: res.data.msg || "编辑失败!",
                                            type: "error"
                                        });
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
  