<template>
  <div>
    <el-container>
      <el-header style="padding: 0px;display:flex;justify-content:space-between;align-items: center">
        <div style="display: inline">
          <el-input
            placeholder="通过手机号或会员卡号查询,记得回车哦..."
            clearable
            @change="keywordsChange"
            style="width: 300px;margin: 0px;padding: 0px;"
            size="mini"
            @keyup.enter.native="searchOrder"
            prefix-icon="el-icon-search"
            v-model="keywords">
          </el-input>
          <el-button type="primary" size="mini" style="margin-left: 5px" icon="el-icon-search" @click="searchOrder">搜索
          </el-button>
        </div>
      </el-header>
      <el-main style="padding-left: 0px;padding-top: 0px">
        <div>
          <el-table
            :data="orders"
            v-loading="tableLoading"
            border
            stripe
            size="mini"
            style="width: 100%">
            <el-table-column
              prop="transaction_name"
              align="left"
              fixed
              label="交易名称"
              width="200">
            </el-table-column>
            <el-table-column
              prop="amount"
              width="200"
              align="left"
              label="交易金额">
            </el-table-column>
            <el-table-column
              prop="phoneNumber"
              width="200"
              align="left"
              label="会员手机号">
            </el-table-column>
            <el-table-column
              prop="name"
              width="200"
              align="left"
              label="会员名称">
            </el-table-column>
            <el-table-column
              align="left"
              label="交易日期">
              <template slot-scope="scope">{{ scope.row.transaction_time | formatDateTime}}</template>
            </el-table-column>
          </el-table>
          <div style="display: flex;justify-content: space-between;margin: 2px">
            <el-pagination
              background
              :page-size="10"
              :current-page="currentPage"
              @current-change="currentChange"
              layout="prev, pager, next"
              :total="totalCount">
            </el-pagination>
          </div>
        </div>
      </el-main>
    </el-container>
  </div>
</template>
<script>
  export default {
    data() {
      return {
        orders: [],
        keywords: '',
        totalCount: -1,
        currentPage: 1,
        tableLoading: false,
      };
    },
    mounted: function () {
      this.initData();
      this.loadOrders()
    },
    methods: {
      keywordsChange(val) {
          this.loadOrders();
      },
      searchOrder() {
        this.loadOrders();
      },
      currentChange(currentChange) {
        this.currentPage = currentChange;
        this.loadOrders();
      },
      loadOrders() {
        var _this = this;
        this.tableLoading = true;
        this.getRequest("/shopping/basic/getTransHistoryByPhone?phoneNumber=" + this.keywords +"&page="+ this.currentPage+"&size=10").then(resp => {
          this.tableLoading = false;
          if (resp && resp.status == 200) {
            var data = resp.data;
            _this.orders = data.orders;
            _this.totalCount = data.count;
          }
        })
      },
      initData() {
        var _this = this;
        this.loadOrders();
      },
      handleChangeNumber(row){
        console.log(row)
      },
    }
  };
</script>
<style>
  .el-dialog__body {
    padding-top: 0px;
    padding-bottom: 0px;
  }

  .slide-fade-enter-active {
    transition: all .8s ease;
  }

  .slide-fade-leave-active {
    transition: all .8s cubic-bezier(1.0, 0.5, 0.8, 1.0);
  }

  .slide-fade-enter, .slide-fade-leave-to {
    transform: translateX(10px);
    opacity: 0;
  }
</style>
