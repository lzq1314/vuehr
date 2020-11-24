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
            :disabled="advanceSearchViewVisible"
            @keyup.enter.native="searchOrder"
            prefix-icon="el-icon-search"
            v-model="keywords">
          </el-input>
          <el-button type="primary" size="mini" style="margin-left: 5px" icon="el-icon-search" @click="searchOrder">搜索
          </el-button>
          <el-button slot="reference" type="primary" size="mini" style="margin-left: 5px"
                     @click="showAdvanceSearchView"><i
            class="fa fa-lg" v-bind:class="[advanceSearchViewVisible ? faangledoubleup:faangledoubledown]"
            style="margin-right: 5px"></i>高级搜索
          </el-button>
        </div>
        <div style="margin-left: 5px;margin-right: 20px;display: inline">
          <el-upload
            :show-file-list="false"
            accept="application/vnd.ms-excel"
            action="/employee/basic/importEmp"
            :on-success="fileUploadSuccess"
            :on-error="fileUploadError" :disabled="fileUploadBtnText=='正在导入'"
            :before-upload="beforeFileUpload" style="display: inline">
            <el-button size="mini" type="success" :loading="fileUploadBtnText=='正在导入'"><i class="fa fa-lg fa-level-up"
                                                                                          style="margin-right: 5px"></i>{{fileUploadBtnText}}
            </el-button>
          </el-upload>
          <el-button type="success" size="mini" @click="exportEmps"><i class="fa fa-lg fa-level-down"
                                                                       style="margin-right: 5px"></i>导出数据
          </el-button>
          <el-button type="primary" size="mini" icon="el-icon-plus"
                     @click="showAddOrderView">
            创建订单
          </el-button>
          <el-button type="primary" size="mini" icon="el-icon-plus"
                     @click="doSettlement">
            结算
          </el-button>
        </div>
      </el-header>
      <el-main style="padding-left: 0px;padding-top: 0px">
        <div>
          <transition name="slide-fade">
            <div
              style="margin-bottom: 10px;border: 1px;border-radius: 5px;border-style: solid;padding: 5px 0px 5px 0px;box-sizing:border-box;border-color: #20a0ff"
              v-show="advanceSearchViewVisible">
              <el-row>
                <el-col :span="5">
                  手机号:
                  <el-input prefix-icon="el-icon-edit" v-model="params.phoneNumber" size="mini" style="width: 150px"
                            placeholder="请输入手机号"></el-input>
                </el-col>
                <el-col :span="11">
                  创建日期:
                  <el-date-picker
                    v-model="params.beginDateScope"
                    unlink-panels
                    size="mini"
                    type="datetimerange"
                    value-format="yyyy-MM-dd HH:mm:ss"
                    range-separator="至"
                    start-placeholder="开始日期"
                    end-placeholder="结束日期">
                  </el-date-picker>
                </el-col>
                <el-col :span="4" :offset="4">
                  <el-button size="mini" @click="cancelSearch">取消</el-button>
                  <el-button icon="el-icon-search" type="primary" size="mini" @click="searchOrder">搜索</el-button>
                </el-col>
              </el-row>
            </div>
          </transition>
          <el-table
            :data="orders"
            v-loading="tableLoading"
            border
            stripe
            @selection-change="handleSelectionChange"
            size="mini"
            style="width: 100%">
            <el-table-column
              type="selection"
              align="left"
              width="30">
            </el-table-column>
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
              width="100"
              label="会员名称">
            </el-table-column>
            <el-table-column
              prop="balance"
              label="余额">
            </el-table-column>
            <el-table-column
              prop="remark"
              width="180"
              align="left"
              label="备注">
            </el-table-column>
            <!-- <el-table-column
              fixed="right"
              label="操作"
              width="195">
              <template slot-scope="scope">
                <el-button @click="showEditMemberView(scope.row)" style="padding: 3px 4px 3px 4px;margin: 2px"
                           size="mini">编辑
                </el-button>
                <el-button @click="addMoneyView(scope.row)" style="padding: 3px 4px 3px 4px;margin: 2px"
                           size="mini">充值
                </el-button>
                <el-button style="padding: 3px 4px 3px 4px;margin: 2px" type="primary"
                           size="mini">查看高级资料
                </el-button>
                <el-button type="danger" style="padding: 3px 4px 3px 4px;margin: 2px" size="mini"
                           @click="deleteEmp(scope.row)">删除
                </el-button>
              </template>
            </el-table-column> -->
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
                <el-form-item label="会员名称:" prop="member_name">
                  <el-input prefix-icon="el-icon-edit" v-model="order.member_name" size="mini" style="width: 150px"
                            placeholder="请输入员工姓名"></el-input>
                </el-form-item>
                <el-form-item label="会员卡号:" prop="memberCardNumber">
                  <el-input prefix-icon="el-icon-edit" v-model="order.memberCardNumber" size="mini" style="width: 150px"
                            placeholder="请输入员工姓名"></el-input>
                </el-form-item>
                <el-form-item label="手机号:" prop="phoneNumber">
                  <el-input prefix-icon="el-icon-edit" v-model="order.phoneNumber" size="mini" style="width: 150px"
                            placeholder="请输入员工姓名"></el-input>
                </el-form-item>
              <el-form-item label="商品名称:" prop="commodity">
                <el-select v-model="order.commodity" style="width: 150px" size="mini" placeholder="请选择民族">
                  <el-option
                    v-for="item in commoditys"
                    :key="item.id"
                    :label="item.commodity_name"
                    :value="item.id">
                  </el-option>
                </el-select>
              </el-form-item>
                <el-form-item label="商品数量:" prop="commodity_count">
                  <el-input prefix-icon="el-icon-edit" v-model="order.commodity_count" size="mini" style="width: 150px"
                            placeholder="请输入员工姓名"></el-input>
                </el-form-item>
          <span slot="footer" class="dialog-footer">
     <el-table
       :data="shops"
       v-loading="tableLoading"
       border
       stripe
       @selection-change="handleSelectionChange"
       size="mini"
       style="width: 80%">
       <el-table-column
         type="selection"
         align="left"
         width="150">
       </el-table-column>
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
         label="选择数量"
         width="195">
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
        members: [],
        keywords: '',
        fileUploadBtnText: '导入数据',
        beginDateScope: '',
        faangledoubleup: 'fa-angle-double-up',
        faangledoubledown: 'fa-angle-double-down',
        dialogTitle: '',
        multipleSelection: [],
        depTextColor: '#c0c4cc',
        commoditys: [],
        politics: [],
        positions: [],
        joblevels: [],
        totalCount: -1,
        currentPage: 1,
        degrees: [{id: 4, name: '大专'}, {id: 5, name: '本科'}, {id: 6, name: '硕士'}, {id: 7, name: '博士'}, {
          id: 3,
          name: '高中'
        }, {id: 2, name: '初中'}, {id: 1, name: '小学'}, {id: 8, name: '其他'}],
        deps: [],
        defaultProps: {
          label: 'name',
          isLeaf: 'leaf',
          children: 'children'
        },
        dialogVisible: false,
        tableLoading: false,
        advanceSearchViewVisible: false,
        showOrHidePop: false,
        showOrHidePop2: false,
        emp: {
          name: '',
          gender: '',
          birthday: '',
          idCard: '',
          wedlock: '',
          nationId: '',
          nativePlace: '',
          politicId: '',
          email: '',
          phone: '',
          address: '',
          departmentId: '',
          departmentName: '所属部门...',
          jobLevelId: '',
          posId: '',
          engageForm: '',
          tiptopDegree: '',
          specialty: '',
          school: '',
          beginDate: '',
          workState: '',
          workID: '',
          contractTerm: '',
          conversionTime: '',
          notWorkDate: '',
          beginContract: '',
          endContract: '',
          sex: '',
          phoneNumber: '',
          telPhoneNumber: '',
          idCardNumber: '',
          memberCardNumber: '',
          remark:''
        },
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
        params:{
          phoneNumber: '',
          beginDateScope:''
        },
        rules: {
          name: [{required: true, message: '必填:姓名', trigger: 'blur'}]
        },
        addMoneyDialogTitle: '',
        addMoneyDialogVisible: false,
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
      fileUploadSuccess(response, file, fileList) {
        if (response) {
          this.$message({type: response.status, message: response.msg});
        }
        this.fileUploadBtnText = '导入数据';
      },
      fileUploadError(err, file, fileList) {
        this.$message({type: 'error', message: "导入失败!"});
        this.fileUploadBtnText = '导入数据';
      },
      beforeFileUpload(file) {
        this.fileUploadBtnText = '正在导入';
      },
      exportEmps() {
        window.open("/employee/basic/exportEmp", "_parent");
      },
      cancelSearch() {
        this.advanceSearchViewVisible = false;
        this.emptyMemberData();
        this.beginDateScope = '';
      },
      showAdvanceSearchView() {
        this.advanceSearchViewVisible = !this.advanceSearchViewVisible;
        this.keywords = '';
        if (!this.advanceSearchViewVisible) {
          this.emptyMemberData();
          this.beginDateScope = '';
        }
      },
      handleSelectionChange(val) {
        this.multipleSelection = val;
      },
      deleteManyEmps() {
        this.$confirm('此操作将删除[' + this.multipleSelection.length + ']条数据, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          var ids = '';
          for (var i = 0; i < this.multipleSelection.length; i++) {
            ids += this.multipleSelection[i].id + ",";
          }
          this.doDelete(ids);
        }).catch(() => {
        });
      },
      deleteEmp(row) {
        this.$confirm('此操作将永久删除[' + row.name + '], 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.doDelete(row.id);
        }).catch(() => {
        });
      },
      keywordsChange(val) {
       // if (val == '') {
          this.loadOrders();
        //}
      },
      searchOrder() {
        this.loadOrders();
      },
      currentChange(currentChange) {
        this.currentPage = currentChange;
        this.loadOrders();
      },
      loadOrders() {

        this.getRequest("/shopping/basic/getOneMemberByPhone?phoneNumber=" + this.keywords).then(resp => {
          if (resp && resp.status == 200) {
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
           // _this.emptyMemberData();
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

               }
             })
          }
          })
      },
      cancelEidt(){
        this.dialogVisible = false;
        this.emptyMemberData();
      },
      cancelMemberMoneyEidt() {
        this.addMoneyDialogVisible= false,
        this.emptyAddMoneyData();
      },
      showDepTree() {
        this.showOrHidePop = !this.showOrHidePop;
      },
      showDepTree2() {
        this.showOrHidePop2 = !this.showOrHidePop2;
      },
      handleNodeClick(data) {
        this.emp.departmentName = data.name;
        this.emp.departmentId = data.id;
        this.showOrHidePop = false;
        this.depTextColor = '#606266';
      },
      handleNodeClick2(data) {
        this.emp.departmentName = data.name;
        this.emp.departmentId = data.id;
        this.showOrHidePop2 = false;
        this.depTextColor = '#606266';
      },
      initData() {
        var _this = this;
      },
      showEditMemberView(row) {
        this.dialogTitle = "编辑会员信息";
        this.member = row;
        this.dialogVisible = true;
      },
      showAddOrderView() {
        if(this.keywords == null || this.keywords ==""){
          this.$alert('请输入会员手机号或会员卡号后查询', '友情提示', {
            confirmButtonText: '确定',
            /* callback: action => {
              this.$notify({
                title: '重要重要!',
                type: 'warning',
                message: '小伙伴们需要注意的是，目前只有权限管理模块完工了，因此这个项目中你无法看到所有的功能，除了权限管理相关的模块。权限管理相关的模块主要有两个，分别是 [系统管理->基础信息设置->权限组] 可以管理角色和资源的关系， [系统管理->操作员管理] 可以管理用户和角色的关系。',
                duration: 0
              });
            } */
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
      emptyMemberData() {
        this.member = {
          id: '',
          sex: '',
          phoneNumber: '',
          telPhoneNumber: '',
          idCardNumber: '',
          memberCardNumber: '',
          remark:''
        }
      },
      emptyAddMoneyData(){
        this.memberMoney = {
          id: '',
          name: '',
          phoneNumber: '',
          memberCardNumber: '',
          amount: ''
        }
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
       /* _this.order.member_id = data.member.id;
        _this.order.phoneNumber = data.member.phoneNumber;
        _this.order.memberCardNumber = data.member.memberCardNumber;
        _this.order.member_name = data.member.name;
        _this.order.balance = data.member.balance;
 */
        var memberName = this.order.member_name
        var memberPhoneNumber = this.order.phoneNumber
        var memberCardNumber = this.order.memberCardNumber
        var balance = this.order.balance
        var amount = 0;//
        for(var i =0;i<orderList.length;i++){
            amount = amount + (orderList[i].commodity_count * orderList[i].commodity_amount);
        }
        //查询用户余额
        /* this.getRequest("/shopping/basic/getOneMemberByPhone?phoneNumber=" + this.keywords).then(resp => {
          this.tableLoading = false;
          if (resp && resp.status == 200) {
            var data = resp.data;
            memberCardNumber = data.member.memberCardNumber;
            balance = data.member.balance;
          }
        }) */
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
                  /* var data = resp.data;
                  _this.orders = data.orders;
                  _this.totalCount = data.count;
                 // _this.emptyMemberData(); */
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
