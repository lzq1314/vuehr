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
              prop="commodity_name"
              align="left"
              fixed
              label="商品名称"
              width="90">
            </el-table-column>
            <el-table-column
              prop="commodity_amount"
              width="80"
              align="left"
              label="商品金额">
            </el-table-column>
            <el-table-column
              prop="commodity_count"
              width="150"
              align="left"
              label="商品数量">
            </el-table-column>
            <el-table-column
              width="150"
              align="left"
              label="创建日期">
              <template slot-scope="scope">{{ scope.row.create_time | formatDateTime}}</template>
            </el-table-column>
            <el-table-column
              prop="phoneNumber"
              width="150"
              label="手机号">
            </el-table-column>
            <el-table-column
              width="150"
              prop="memberCardNumber"
              label="会员卡号">
            </el-table-column>
            <el-table-column
              prop="member_name"
              label="会员名称">
            </el-table-column>
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
        this.getRequest("/shopping/basic/getOrderHistoryByPhone?phoneNumber=" + this.keywords + "&memberCardNumber="+ this.keywords+"&page="+ this.currentPage+"&size=10").then(resp => {
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
      }
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
