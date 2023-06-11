<template>
  <div style="margin-top: 30px">
    <el-card class="welcome-card note50" style="width: 49%; margin: 0; font-size: 14px">
      <div slot="header" class="clearfix">
        <span style="font-size: 16px">服务器</span>
      </div>
      <div class="text item">环境变量：{{ serverInfo.EnvironmentName }}</div>
      <div class="text item">系统架构：{{ serverInfo.OSArchitecture }}</div>
      <div class="text item">
        ContentRootPath：{{ serverInfo.ContentRootPath }}
      </div>
      <div class="text item">WebRootPath：{{ serverInfo.WebRootPath }}</div>
      <div class="text item">
        .NET Core版本：{{ serverInfo.FrameworkDescription }}
      </div>
      <div class="text item">内存占用：{{ serverInfo.MemoryFootprint }}</div>
      <div class="text item">启动时间：{{ serverInfo.WorkingTime }}</div>
      <div>
        <br />
      </div>
    </el-card>
  </div>
</template>

<script>
// import Vue from "vue";
// import VCharts from "v-charts";
// Vue.use(VCharts);

import applicationUserManager from "../Auth/applicationusermanager";
import {
  getServerInfo,
  getAccessLogs,
  getIds4UsersGrow,
  getActiveUsers,
  getAchieveUsers_IS4,
} from "../api/api";

export default {
  name: "Welcome",
  data() {
    return {
      listLoading: false,
      welcomeInitData: {
        activeUsers: [],
        activeUserCount: 0,
        logs: [],
        errorCount: 0,
      },
      serverInfo: {},
      extend: {
        series: {
          label: {
            normal: {
              show: true,
            },
          },
        },
      },
      lineChartDataIDS4: {
        columns: [],
        rows: [],
        today: 0,
      },
      lineChartSettings7Day: {
        metrics: ["count"],
        dimension: ["date"],
      },
      lineChartMarkPoint: {
        data: [
          {
            name: "最大值",
            type: "max",
          },
          {
            name: "最小值",
            type: "min",
          },
        ],
      },
    };
  },
  methods: {
    getTypeName(count) {
      if (count >= 10 && count < 50) {
        return "warning";
      }
      if (count < 10) {
        return "primary";
      }
      return "danger";
    },
    getBck(index) {
      return `background: rgb(${43 + index * 14}, ${148 + index * 7
        }, 255) none repeat scroll 0% 0%;`;
    },
    toLogs() {
      this.$router.replace({
        path: "/Logs/Index",
      });
    },
  },
  mounted() {
    var curTime = new Date();
    if (window.localStorage.TokenExpire) {
      var expiretime = new Date(Date.parse(window.localStorage.TokenExpire));
      if (curTime >= expiretime) {
        if (global.IS_IDS4) {
          applicationUserManager.login();
        } else {
          this.$router.push("/login");
        }
      }
    } else {
      if (global.IS_IDS4) {
        applicationUserManager.login();
      } else {
        this.$router.push("/login");
      }
    }
    getServerInfo({}).then((res) => {
      this.serverInfo = res.data.response;
    });
  },
};
</script>

<style scoped>
.amazing /deep/ .el-badge__content {
  background-color: purple !important;
}

.bg-blue-sub-item {
  max-width: 120px !important;
  height: 50px;
  float: left;
  color: #fff;
  font-size: 12px;
  line-height: 50px;
  padding: 0 10px;
}

.bg-blue-sub-item .acc-user {
  max-width: 115px;
  display: block;
  overflow: hidden;
  text-overflow: ellipsis;
}

.bg-blue-sub-item-m {
  height: 50px;
  float: left;
  color: #fff;
  font-size: 12px;
  line-height: 50px;
  padding: 0 10px;
}

.bg-blue-sub-item-m .acc-user {
  display: block;
  overflow: hidden;
  text-overflow: ellipsis;
}

.note .text {
  font-size: 14px;
}

.note .item {
  margin-bottom: 18px;
}
</style>

<style scoped>
.panel-group {
  margin-bottom: 18px;
  margin-right: 2%;
}

.card-panel-col {
  /* margin-bottom: 32px; */
  width: 113px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  margin-left: 15px;
  float: right;
}

.card-panel {
  height: 108px;
  font-size: 12px;
  position: relative;
  overflow: hidden;
  color: #666;
  background: #fff;
  box-shadow: 4px 4px 40px rgba(0, 0, 0, 0.05);
  border-color: rgba(0, 0, 0, 0.05);
}

.card-panel .card-panel-icon-wrapper {
  color: #fff;
}

.card-panel .card-panel .icon-people {
  background: #40c9c6;
}

.card-panel .card-panel .icon-message {
  background: #36a3f7;
}

.card-panel .card-panel .icon-money {
  background: #f4516c;
}

.card-panel .card-panel .icon-shopping {
  background: #34bfa3;
}

.card-panel .icon-people {
  color: #40c9c6;
}

.card-panel .icon-message {
  color: #36a3f7;
}

.card-panel .icon-money {
  color: #f4516c;
}

.card-panel .icon-shopping {
  color: #34bfa3;
}

.card-panel .card-panel-icon-wrapper {
  float: left;
  margin: 14px 0 0 14px;
  padding: 16px;
  transition: all 0.38s ease-out;
  border-radius: 6px;
}

.card-panel .card-panel-icon {
  float: left;
  font-size: 48px;
}

.card-panel .card-panel-description {
  float: left;
  font-weight: bold;
  margin-left: 30px;
  margin-top: 20px;
}

.card-panel .card-panel-description .card-panel-text {
  line-height: 18px;
  color: rgba(0, 0, 0, 0.45);
  font-size: 16px;
  margin-bottom: 12px;
}

.card-panel .card-panel-description .card-panel-num {
  font-size: 36px;
  color: #f4516c;
}

.extoday {
  cursor: pointer;
}

.card-acuser-num {
  font-size: 36px;
  color: #40c9c6;
}

.bg-blue-sub-item /deep/ .el-badge__content.is-fixed {
  top: 5px !important;
}

.bg-blue-sub-item-m /deep/ .el-badge__content.is-fixed {
  top: 5px !important;
}

@media (max-width: 550px) {
  .note50 {
    width: 100% !important;
  }

  .statistical-cus {
    width: 100%;
  }

  .card-panel-col {
    margin-bottom: 5px !important;
    width: 100% !important;
  }

  .card-panel-icon-wrapper {
    float: none !important;
    width: 100%;
    height: 100%;
    margin: 0 !important;
  }

  .card-panel-icon-wrapper .svg-icon {
    display: block;
    margin: 14px auto !important;
    float: none !important;
  }
}
</style>
