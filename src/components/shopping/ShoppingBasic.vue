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
        <div style="margin-left: 5px;margin-right: 20px;display: inline">
          <el-button type="primary" size="mini" icon="el-icon-plus"
                     @click="showAddOrderView">
            创建订单
          </el-button>
          <el-button type="success" size="mini" icon="el-icon-plus"
                     @click="doSettlement">
            结算
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
    <el-form :model="order" :rules="rules" ref="addOrderForm" style="margin: 0px;padding: 0px;">
      <div style="text-align: left">
        <el-dialog
          :title="dialogTitle"
          style="padding: 0px;"
          :close-on-click-modal="false"
          :visible.sync="dialogVisible"
          width="77%">
          <el-row>
            <el-col :span="6">
              <div>
                <el-form-item label="会员名称:" prop="member_name">
                  <el-input :disabled="true" v-model="order.member_name" size="mini" style="width: 150px"
                            placeholder="请输入员工姓名"></el-input>
                </el-form-item>
              </div>
             </el-col>
             <el-col :span="6">
               <div>
                <el-form-item label="会员卡号:" prop="memberCardNumber">
                  <el-input :disabled="true" v-model="order.memberCardNumber" size="mini" style="width: 150px"
                            placeholder="请输入员工姓名"></el-input>
                </el-form-item>
              </div>
            </el-col>
            <el-col :span="6">
              <div>
                <el-form-item label="手机号:" prop="phoneNumber">
                  <el-input :disabled="true" v-model="order.phoneNumber" size="mini" style="width: 150px"
                            placeholder="请输入员工姓名"></el-input>
                </el-form-item>
              </div>
            </el-col>
          </el-row>
          <span slot="footer" class="dialog-footer">
     <el-table
       :data="shops"
       v-loading="tableLoading"
       border
       stripe
       size="mini"
       style="width: 80%">
       <el-table-column
         prop="commodity_name"
         align="left"
         fixed
         label="商品名称"
         width="150">
       </el-table-column>
       <el-table-column
         prop="commodity_amount"
         width="150"
         align="left"
         label="商品金额">
       </el-table-column>
       <el-table-column
         prop="cnt"
         width="150"
         align="left"
         label="商品数量">
       </el-table-column>
       <el-table-column
         fixed="right"
         label="选择数量">
         <template slot-scope="scope">
           <el-input-number v-model="scope.row.cnt" @change="handleChangeNumber(scope.row)" :min="0" :max="100" label="描述文字"></el-input-number>
         </template>
       </el-table-column>
     </el-table>
    <el-button size="mini" @click="cancelEidt">取 消</el-button>
    <el-button size="mini" type="primary" @click="addOrder('addOrderForm')">确 定</el-button>
  </span>
        </el-dialog>
      </div>
    </el-form>
  </div>
</template>
<script>
  export default {
    data() {
      return {
        shops: [],
        orders: [],
        keywords: '',
        dialogTitle: '',
        commoditys: [],
        totalCount: -1,
        currentPage: 1,
        dialogVisible: false,
        tableLoading: false,
        order: {
          id: '',
          commodity: '',
          member_id: '',
          commodity_name: '',
          commodity_amount: '',
          phoneNumber: '',
          memberCardNumber:'',
          commodity_count:'',
          member_name:'',
          balance: 0
        },
        rules: {
          name: [{required: true, message: '必填:姓名', trigger: 'blur'}]
        },
        memberMoney: {
          id: '',
          name: '',
          phoneNumber: '',
          memberCardNumber: '',
          amount: ''
        }
      };
    },
    mounted: function () {
      this.initData();
    },
    methods: {
      keywordsChange(val) {
          this.loadOrders();
      },
      searchOrder() {
        if(this.keywords == null || this.keywords ==""){
          this.$alert('请输入会员手机号或会员卡号后查询', '友情提示', {
            confirmButtonText: '确定',
          });
          return
        }
        this.loadOrders();
      },
      currentChange(currentChange) {
        this.currentPage = currentChange;
        this.loadOrders();
      },
      loadOrders() {
        this.getRequest("/shopping/basic/getOneMemberByPhone?phoneNumber=" + this.keywords).then(resp => {
          if (resp && resp.status == 200) {
            if(resp.data.member == null){
              this.$alert('当前用户不是会员，请先创建会员', '友情提示', {
                confirmButtonText: '确定',
              });
              return;
            }
            var data = resp.data;
            _this.order.member_id = data.member.id;
            _this.order.phoneNumber = data.member.phoneNumber;
            _this.order.memberCardNumber = data.member.memberCardNumber;
            _this.order.member_name = data.member.name;
            _this.order.balance = data.member.balance;
            _this.shops = data.commodityList;
          }
        })
        var _this = this;
        this.tableLoading = true;
        this.getRequest("/shopping/basic/getOrderByPhone?phoneNumber=" + this.keywords + "&memberCardNumber="+ this.keywords).then(resp => {
          this.tableLoading = false;
          if (resp && resp.status == 200) {
            var data = resp.data;
            _this.orders = data.orders;
            _this.totalCount = data.count;
          }
        })
      },
      addOrder(formName) {
        var _this = this;
        this.$refs[formName].validate((valid) => {
          if (valid) {
              //添加
              var par = {
                commodity: this.shops,
                phoneNumber: this.keywords
              }
             this.tableLoading = true;
             this.postRequestJson("/shopping/basic/addOrder",par).then(resp => {
               _this.tableLoading = false;
               if (resp && resp.status == 200) {
                 this.dialogVisible = false;
                 this.loadOrders();
               }
             })
          }
          })
      },
      cancelEidt(){
        this.dialogVisible = false;
        this.emptyMemberData();
      },
      initData() {
        var _this = this;
      },
      showAddOrderView() {
        if(this.keywords == null || this.keywords ==""){
          this.$alert('请输入会员手机号或会员卡号后查询', '友情提示', {
            confirmButtonText: '确定',
          });
          return
        }
        this.dialogTitle = "加购商品";
        this.dialogVisible = true;
        var _this = this;

      this.getRequest("/shopping/basic/getCommodity").then(resp => {
                if (resp && resp.status == 200) {
                  var data = resp.data;
                  this.commoditys = data.commodity;
                }
              })

      },
      handleChangeNumber(row){
        console.log(row)
      },
      doSettlement(){
        var orderList = this.orders;
        if(orderList.length == 0){
          this.$alert('没有消费内容，无要结算信息', '友情提示', {
            confirmButtonText: '确定',
          });
          return;
        }
        //会员名称，手机号，会员卡号，余额,消费金额
        var memberName = this.order.member_name
        var memberPhoneNumber = this.order.phoneNumber
        var memberCardNumber = this.order.memberCardNumber
        var balance = this.order.balance
        var amount = 0;//
        for(var i =0;i<orderList.length;i++){
            amount = amount + (orderList[i].commodity_count * orderList[i].commodity_amount);
        }
        if(amount > balance){
          this.$alert(memberName+' 会员余额不足，当前余额：'+balance+'，消费金额：'+amount+'；请及时充值！！！', '友情提示', {
            confirmButtonText: '确定',
          });
          return;
        }else{
          this.$alert(memberName+' 会员，手机号：'+memberPhoneNumber+'，会员卡号：'+memberCardNumber+'，当前余额：'+balance+',消费金额：'+amount+';核对无误后点击确定结算！！！', '友情提示', {
            confirmButtonText: '确定',
            callback: action => {
              this.getRequest("/shopping/basic/doSettlement?phoneNumber=" + this.keywords + "&amount="+ amount).then(resp => {
                this.tableLoading = false;
                if (resp && resp.status == 200) {
                  this.loadOrders();
                }
              })
            }
          });
        }

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
