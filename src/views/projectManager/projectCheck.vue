<template>
  <div class="app-container">
    <el-form ref="queryForm" :model="queryParams" :inline="true" size="small" style="margin-top:10px;">
      <el-form-item label="项目名称" prop="itemName">
        <el-input
          v-model="queryParams.itemName"
          placeholder="请输入项目名称"
          clearable
          style="width: 140px"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="项目用途" prop="useType">
        <el-select v-model="queryParams.useType" placeholder="请选择项目用途" style="width: 150px">
          <el-option label="非税收入" value="非税收入" />
          <el-option label="医疗项目" value="医疗项目" />
          <el-option label="其他项目" value="其他项目" />
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" @click="handleQuery">搜索</el-button>
        <el-button icon="el-icon-refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          type="primary"
          size="small"
          @click="handleMultCheck"
        >批量审核</el-button>
      </el-col>
    </el-row>

    <el-table :data="projectList" style="width: 100%;margin-top:30px;" border @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column align="center" label="状态" prop="isEnable">
        <template slot-scope="scope">
          <el-tag
            :type="scope.row.isEnable ? 'success' : 'info'"
            disable-transitions
          >{{ scope.row.isEnable ? '已完成' : '待审核' }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column align="center" label="项目业务码" prop="itemCode" />
      <el-table-column align="center" label="项目名称" prop="itemName" :show-overflow-tooltip="true" />
      <el-table-column align="center" label="项目生效日期" prop="itemEffdate" />
      <el-table-column align="center" label="项目失效日期" prop="itemExpdate" />
      <el-table-column align="center" label="资金性质" prop="fundsnatureCode" />
      <el-table-column align="center" label="操作" width="220">
        <template slot-scope="scope">
          <el-button type="success" size="mini" @click="handleLook(scope.row)">查看</el-button>
          <el-button type="primary" size="mini" @click="handleCheck(scope)">审核</el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
      style="margin-top:20px;float:right;margin-right:20px;"
      center
      background
      margin-top="10"
      layout="prev, pager, next, sizes, total, jumper"
      :page-sizes="[10,20,50,100]"
      :page-size="queryParams.limit"
      :total="queryParams.total"
      :current-page="queryParams.page"
      @current-change="handleCurrentChange"
      @size-change="handleSizeChange"
    />

    <el-dialog :visible.sync="dialogVisible" :title="dialogType==='edit'?'项目变动':'新增项目'">
      <el-form ref="project" :model="project" :rules="rules" label-width="80px" label-position="right" style="padding-right:25px;">
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="项目编码" :label-width="formLabelWidth" prop="itemCode">
              <el-input v-model="project.itemCode" placeholder="项目编码" />
            </el-form-item>
            <el-form-item label="生效日期" :label-width="formLabelWidth">
              <el-date-picker v-model="project.itemEffdate" type="date" placeholder="选择日期" style="width: 100%;" />
            </el-form-item>
            <el-form-item label="收入类别" :label-width="formLabelWidth" prop="incomeSortCode">
              <!-- <el-input v-model="project.incomeSortCode" placeholder="收入类别" /> -->
              <el-select v-model="project.incomeSortCode" placeholder="根据实际情况选择">
                <el-option label="直缴" value="直缴" />
                <el-option label="汇缴" value="汇缴" />
                <el-option label="代缴" value="代缴" />
              </el-select>
            </el-form-item>
            <el-form-item label="收缴方式" :label-width="formLabelWidth">
              <el-select v-model="project.paymode" placeholder="请选择收缴方式">
                <el-option label="直缴" value="直缴" />
                <el-option label="汇缴" value="汇缴" />
                <el-option label="代缴" value="代缴" />
              </el-select>
            </el-form-item>
            <el-form-item label="资金性质" :label-width="formLabelWidth" prop="fundsnatureCode">
              <!-- <el-input v-model="project.fundsnatureCode" placeholder="资金性质" /> -->
              <el-select v-model="project.fundsnatureCode" placeholder="根据实际情况选择">
                <el-option label="直缴" value="直缴" />
                <el-option label="汇缴" value="汇缴" />
                <el-option label="代缴" value="代缴" />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="项目名称" :label-width="formLabelWidth" prop="itemName">
              <el-input v-model="project.itemName" placeholder="项目名称" />
            </el-form-item>
            <el-form-item label="失效日期" :label-width="formLabelWidth">
              <!-- <el-input v-model="project.itemExpdate" placeholder="失效日期" /> -->
              <el-date-picker v-model="project.itemExpdate" type="date" placeholder="选择日期" style="width: 100%;" />
            </el-form-item>
            <el-form-item label="助记码" :label-width="formLabelWidth" prop="mnen">
              <el-input v-model="project.mnen" placeholder="助记码" />
            </el-form-item>
            <el-form-item label="预算科目" :label-width="formLabelWidth">
              <el-input v-model="project.subject" placeholder="根据实际情况填写" />
            </el-form-item>
            <el-form-item label="备注" :label-width="formLabelWidth">
              <el-input v-model="project.note" placeholder="备注" />
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <div style="text-align:right;">
        <el-button type="danger" @click="cancel">取消</el-button>
        <el-button type="primary" @click="confirmRole">确认</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
// import { listRole, addRole, updateRole } from '@/api/projectManager'

export default {
  data () {
    return {
    //   loading: true,
      queryParams: { // 查询参数
        itemName: '',
        useType: '',
        isEnable: '',
        page: 1,
        limit: 10,
        total: 0
      },
      projectList: [
        {
          isEnable: 1,
          itemCode: '1',
          itemName: '政府基金',
          itemEffdate: '2020-01-02',
          itemExpdate: '2020-02-03',
          fundsnatureCode: '基金收入'
        },
        {
          isEnable: 1,
          itemCode: '2',
          itemName: '政府基金',
          itemEffdate: '2020-01-02',
          itemExpdate: '2020-02-03',
          fundsnatureCode: '基金收入'
        },
        {
          isEnable: 1,
          itemCode: '3',
          itemName: '政府基金',
          itemEffdate: '2020-01-02',
          itemExpdate: '2020-02-03',
          fundsnatureCode: '基金收入'
        },
        {
          isEnable: 1,
          itemCode: '4',
          itemName: '政府基金',
          itemEffdate: '2020-01-02',
          itemExpdate: '2020-02-03',
          fundsnatureCode: '基金收入'
        }
      ],
      project: {},
      dialogVisible: false,
      dialogType: 'new',
      formLabelWidth: '100px',
      multiple: true, // 非多个禁用
      pagesize: 5,
      currpage: 1,
      fileList: [],
      rules: {
        itemCode: [
          { required: true, message: '项目编码不能为空', trigger: 'blur' }
        ],
        itemName: [
          { required: true, message: '项目名称不能为空', trigger: 'blur' }
        ],
        mnen: [
          { required: true, message: '助记码不能为空', trigger: 'blur' }
        ],
        incomeSortCode: [
          { required: true, message: '收入类别不能为空', trigger: 'blur' }
        ],
        fundsnatureCode: [
          { required: true, message: '资金性质不能为空', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getTableData()
  },
  methods: {
    // 获取资源列表
    async getTableData () {
      // this.loading = true
      // const res = await listRole(this.queryParams)
      // this.projectList = res.data.items
      // this.queryParams.total = res.data.total
      // this.queryParams.limit = res.data.limit
      // this.queryParams.page = res.data.page
      // this.selectedList = []
      // this.loading = false
    },
    // 搜索
    handleQuery () {
      this.queryParams.page = 1
      this.getTableData()
    },
    // 表单重置封装
    resetForm (refName) {
      if (this.$refs[refName]) {
        this.$refs[refName].resetFields()
      }
    },
    // 重置
    resetQuery () {
      // this.resetForm('queryParams')
      this.queryParams = {}
    },
    // 多选框选中数据
    handleSelectionChange (selection) {
      this.selectedList = selection
      // this.ids = selection.map(item => item.itemName)
      // console.log(this.ids)
      // this.single = selection.length !== 1
      // this.multiple = !selection.length
    },
    // 批量审核
    handleMultCheck () {

    },
    // 审核按钮
    handleCheck () {

    },
    // 查看按钮
    handleLook (rowData) {
      this.dialogVisible = true
      this.dialogType = 'edit'
      this.project = Object.assign({}, rowData)
    },
    // 模态框取消
    cancel () {
      this.dialogVisible = false
      this.resetForm('project')
    },
    // 模态框提交
    confirmRole () {
    //   this.$refs['project'].validate(valid => {
    //     if (valid) {
    //       if (this.dialogType !== 'edit') { // 新增
    //         addRole(this.role).then(res => {
    //           this.role = res.data
    //           console.log(this.role)
    //           if (res.code === 200) {
    //             this.$set(this.role, {})
    //             this.getTableData() // 重新渲染数据列表
    //             this.dialogVisible = false // 关闭模态框
    //             // this.msgSuccess(res.message)
    //             this.$message({
    //               showClose: true,
    //               message: '添加成功',
    //               type: 'success'
    //             })
    //           } else {
    //             this.$message({
    //               showClose: true,
    //               message: '添加失败',
    //               type: 'error'
    //             }) // 或者弹出后台返回错误
    //           }
    //         })
    //       } else { // 变动
    //         this.role = {
    //           id: this.row.id,
    //           name: this.row.name
    //         }
    //         updateRole(this.role.id, this.role).then(res => {
    //           this.msgSuccess(res.message)
    //           this.getTableData()
    //           this.dialogVisible = false
    //         })
    //       }
    //     }
    //   })
    },
    // 分页
    handleCurrentChange (cpage) {
      // userList.slice((currpage - 1) * pagesize, currpage * pagesize)
      this.currpage = cpage
    },
    handleSizeChange (psize) {
      this.pagesize = psize
    }
  }
}
</script>

<style lang="scss" scoped>
.app-container {
  .roles-table {
    margin-top: 30px;
  }
  .permission-tree {
    margin-bottom: 30px;
  }
}
</style>
