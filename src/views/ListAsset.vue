<template>
  <div>
    根据工号查询：
    <el-input v-model="jobNumber" style="width: 20%" placeholder="请输入工号" @keyup.enter.native = "findByJobNumber">根据工号查询</el-input>
    根据资产编号查询：
    <el-input v-model="assetNumber" style="width: 20%" placeholder="请输入资产编号" @keyup.enter.native = "findByAssetNumber">根据资产编号查询</el-input>
    <br><br>
    <el-button @click="bathAddAsset" type="success"
               :disabled="this.selectionList.length == 0">入库</el-button>
    &nbsp
    <el-button @click="bathDeleteAsset" type="danger"
               :disabled="this.selectionList.length == 0">删除</el-button>
    <br><br>
    <el-table :data="tableData" border style="width: 100%" @selection-change="selectionChange">
      <el-table-column type="selection" width="55">
      </el-table-column>
      <el-table-column fixed type="index" label="序号" width="100">
      </el-table-column>
      <el-table-column prop="jobNumber" label="工号" width="150">
      </el-table-column>
      <el-table-column prop="deviceName" label="资产类型" width="180">
      </el-table-column>
      <el-table-column prop="assetNumber" label="设备编号" width="180">
      </el-table-column>
      <el-table-column prop="status" width="180">
        <template slot="header" slot-scope="scope">
          <el-dropdown @command="listByStatus">
            <span class="el-dropdown-link">
              是否入库<i class="el-icon-arrow-down el-icon--right"></i>
            </span>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item command="是">是</el-dropdown-item>
              <el-dropdown-item command="否">否</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </template>
      </el-table-column>
      <el-table-column fixed="right" label="操作">
        <template slot-scope="scope">
          <el-button @click="addAssetToOA(scope.row)" type="info" size="small"icon="el-icon-upload">入库</el-button>
          <el-button @click="handleEdit(scope.$index,scope.row)" type="info" size="small" icon="el-icon-edit">编辑</el-button>
          <el-button @click="deleteAsset(scope.row)" type="danger" size="small"icon="el-icon-delete">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
            background
            layout="prev, pager, next"
            :page-size= "pageSize"
            :total="total"
            :current-page="currentPage"
            @current-change="pageChange">
    </el-pagination>
    <!-- ’编辑‘弹框界面-->
    <el-dialog :visible.sync="dialogFormVisible" :close-on-click-modal="false" style="width: 100%" title="编辑资产信息" >
      <el-form :model="editForm" ref="editForm">
        <el-form-item style="width: 80%" label="资产id" prop="id" >
          <el-input v-model="editForm.id" ></el-input>
        </el-form-item>
        <el-form-item style="width: 80%" label="工号">
          <el-input v-model="editForm.jobNumber" ></el-input>
        </el-form-item>
        <el-form-item style="width: 80%" label="设备名称" >
          <el-input v-model="editForm.deviceName"></el-input>
        </el-form-item>
        <el-form-item style="width: 80%" label="设备编号" >
          <el-input v-model="editForm.assetNumber"></el-input>
        </el-form-item>
        <el-form-item style="width: 80%" label="是否入库" >
          <el-input v-model="editForm.status"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button @click="resetForm('editForm')">重置</el-button>
        <el-button type="primary" @click="updateAsset">确 定</el-button>
      </div>
    </el-dialog>
  </div>

</template>

<script>
  export default {
    inject: ['reload'], //依赖注入
    methods: {
      pageChange(currentPage){
        const that = this
        that.currentPage = currentPage;
        that.listByStatus("否");
        // that.getList(that)
      },
      //批量选择时触发
      selectionChange(val){
        this.selectionList = val
      },
      // getList(that){
      //   axios.get('http://localhost:8090/listAsset/'+that.currentPage+'/10').then(function (response) {
      //     that.tableData = response.data.content
      //     that.total = response.data.totalElements
      //   })
      // },
      listByStatus(command){
        const that = this
        axios.request({
          url: this.baseUrl+'/listByStatus',
          params:{
            status:command,
            page:that.currentPage,
            size:10
          }
        }).then((response) =>{
          that.tableData = response.data.content
          that.total = response.data.totalElements
        })
      },
      findByJobNumber(){
        let i = parseInt(this.jobNumber)
        const that = this
        axios.get(this.baseUrl+'/findByJobNumber?jobNumber='+i).then((response)=>{
          that.tableData = response.data
          that.total = response.data.totalElements
        })
      },
      findByAssetNumber(){
        let str = this.assetNumber
        axios.get(this.baseUrl+'/findByAssetNumber?assetNumber='+str).then((response)=>{
          this.tableData = response.data
          this.total = response.data.totalElements
        })
      },
      handleEdit(index,row){
        this.dialogFormVisible = true;
        this.editForm = Object.assign({},row);
      },
      updateAsset(){
        let para = Object.assign({},this.editForm);
        axios.post(this.baseUrl+'/updateAsset',para).then((response)=>{
          if(response.data.success == true){
            this.$alert('资产信息修改成功!','提示', {
              confirmButtonText: '确定',
              callback: action => {
                this.reload();
              }
            });
          }
        })
      },
      resetForm(formName) {
        this.$refs[formName].resetFields();
      },
      deleteAsset(row){
        this.$confirm('此操作将永久删除所选中信息, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          axios.delete(this.baseUrl+'/deleteAsset/?id='+row.id)
          this.$alert('删除成功!','提示', {
            confirmButtonText: '确定',
            callback: action => {
              this.reload();
            }
          });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });
        });
      },
      bathDeleteAsset() {
        let assetIds = this.selectionList.map(item => item.id).join()
        this.$confirm('此操作将永久删除所选中信息, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          axios.post(this.baseUrl+'/bathDeleteAsset/?assetIds=' + assetIds)
          this.$alert('删除成功!','提示', {
            confirmButtonText: '确定',
            callback: action => {
              this.reload(); //调用注入的方法达到刷新router-view的目的
              // window.location.reload()
            }
          });
          this.listByStatus("否");
          }).catch(() => {
            this.$message({
              type: 'info',
              message: '已取消删除'
            });
          });
      },
      addAssetToOA(row){
        this.$confirm('确认录入系统', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
        }).then(({ value }) => {
          axios.post(this.baseUrl+'/addAsset/?ids=' + row.id).then(response=> {
            if(response.data.code === 1){
              // this.$message({
              //   type: 'success',
              //   message: '入库成功!'
              // })
              this.$alert('入库成功!','提示', {
                confirmButtonText: '确定',
                callback: action => {
                  this.reload(); //调用注入的方法达到刷新router-view的目的
                  // window.location.reload()
                }
              });
              axios.post(this.baseUrl+'/updateAssetStatus/?ids=' + row.id)
            }else{
              this.$confirm(response.data.msg, '入库失败', {
                confirmButtonText: '确定',
                type: 'warning'
              })
              // console.log(response.data.msg)
            }
          })
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '取消入库'
          });
        });
        },
      bathAddAsset(){
        let ids = this.selectionList.map(item => item.id).join()
        this.$confirm('此操作将所选中信息入库, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          axios.post(this.baseUrl+'/addAsset/?ids=' + ids).then(response=> {
            if(response.data.code === 1){
              // this.$message({
              //   type: 'success',
              //   message: '入库成功!'
              // })
              this.$alert('入库成功!','提示', {
                confirmButtonText: '确定',
                callback: action => {
                  this.reload(); //调用注入的方法达到刷新router-view的目的
                  // window.location.reload()
                }
              });
              axios.post(this.baseUrl+'/updateAssetStatus/?ids=' + ids)
            }else{
              this.$confirm(response.data.msg, '入库失败', {
                confirmButtonText: '确定',
                type: 'warning'
              })
              // console.log(response.data.msg)
            }
          })
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消入库'
          });
        });
      },
    },
    created() {
      const this1 = this
      // this1.getList(this1)
      this1.listByStatus('否');
    },
    data() {
      return {
        baseUrl: 'http://10.2.10.22:8090',
        input:'',
        jobNumber:'',
        assetNumber:'',
        selectionList:'',
        pageSize: 10,
        total: 0,
        tableData: [],
        currentPage: 1,
        checked: true,
        dialogFormVisible: false,
        editForm: {
            id:'',
            jobNumber:'',
            deviceName: '',
            assetNumber:'',
            status: ''
        },
      }
    }
  }
</script>
