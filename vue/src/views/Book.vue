<template>
  <div style="padding : 10px">
    <!--    功能区域-->
    <div style="margin: 10px 0">
      <el-button type="primary" @click="add" v-if="user.role === 1">新增</el-button>
      <el-popconfirm v-if="user.role === 1" title="确定删除吗？" @confirm="deleteBatch">
        <template #reference>
          <el-button type="danger" v-if="user.role === 1">批量删除</el-button>
        </template>
      </el-popconfirm>

    </div>

    <!--      搜索区域-->
    <div style="margin: 10px 0">
      <el-input v-model="search" placeholder="请输入关键字" style="width: 20%" clearable></el-input>
      <el-button type="primary" style="margin-left: 5px" @click="load">查询</el-button>
    </div>

    <el-table :data="tableData" border stripe style="width: 100%" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55"></el-table-column>
      <el-table-column prop="id" label="ID"  sortable> </el-table-column>
      <el-table-column prop="name" label="名称" > </el-table-column>
      <el-table-column prop="price" label="单价"> </el-table-column>
      <el-table-column prop="author" label="作者"> </el-table-column>
      <el-table-column prop="createTime" label="出版时间"> </el-table-column>
      <el-table-column
          label="封面">
        <template #default="scope">
          <el-image
              style="width: 100px; height: 100px"
              :src="scope.row.cover"
              :preview-src-list="[scope.row.cover]">
          </el-image>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="240">
        <template #default="scope">
          <el-button type="text" @click="buy(scope.row.id)">购买</el-button>
          <el-button @click="handleEdit(scope.row)" type="text" v-if="user.role === 1">编辑</el-button>
          <el-popconfirm title="这一段内容确定删除吗？" @confirm="handleDelete(scope.row.id)" v-if="user.role === 1">
            <template #reference>
              <el-button type="text" >删除</el-button>
            </template>
          </el-popconfirm>
        </template>
      </el-table-column>
    </el-table>

    <div style="margin: 10px 0">
      <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="currentPage"
          :page-sizes="[5, 10, 20]"
          :page-size="pageSize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total">
      </el-pagination>

      <el-dialog title="提示" v-model="dialogVisible" width="30%">
        <el-form :model="form" label-width="120px">
          <el-form-item label="名称">
            <el-input v-model="form.name" style="width: 80%"></el-input>
          </el-form-item>
          <el-form-item label="价格">
            <el-input v-model="form.price" style="width: 80%"></el-input>
          </el-form-item>
          <el-form-item label="作者">
            <el-input v-model="form.author" style="width: 80%"></el-input>
          </el-form-item>
          <el-form-item label="出版时间">
            <el-date-picker v-model="form.createTime" value-format="YYYY-MM-DD" type="date" style="width: 80%" clearable></el-date-picker>
          </el-form-item>
          <el-form-item label="封面">
            <el-upload ref="upload" :action="filesUploadUrl" :on-success="filesUploadSuccess">
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-form-item>
        </el-form>
        <template #footer>
          <span class="dialog-footer">
            <el-button @click="dialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="save" >确 定</el-button>
          </span>
        </template>
      </el-dialog>

    </div>
  </div>
</template>

<script>

import request from "../utils/request";
import {ElMessage} from 'element-plus';

export default {
  name: 'Book',
  components: {},
  data(){
    return {
      user:{},
      loading: true,
      form:{},
      dialogVisible:false,
      search: '',
      currentPage: 1,
      pageSize:10,
      total: 0,
      tableData:[],
      filesUploadUrl: "http://" + window.server.filesUploadUrl + ":9090/files/upload",
      ids: []
    }
  },
  created() {
    let userStr = sessionStorage.getItem("user") || "{}"
    this.user = JSON.parse(userStr)
    // 请求服务端，确认当前登录用户的 合法信息
    request.get("/user/" + this.user.id).then(res => {
      if (res.code === '0') {
        this.user = res.data
      }
    })
    this.load()
  },
  methods:{
    buy(bookId) {
      request.get("/order/buy/" + bookId).then(res => {
        // 请求成功跳转沙箱支付的页面
        window.open(res.data)
      })
    },
    deleteBatch() {
      if (!this.ids.length) {
        ElMessage.warning("请选择数据！")
        return
      } request.post("/book/deleteBatch", this.ids).then(res => {
        if(res.code ==='0'){
          ElMessage.success("批量删除成功")
        } else {
          ElMessage.error("批量删除失败")
        }
      })
    },
    handleSelectionChange(val) {
      this.ids = val.map(v => v.id)
    },
    filesUploadSuccess(res) {
      console.log(res)
      this.form.cover = res.data
    },
    load(){
      this.loading = true
      request.get("/book", {
        params:{
          pageNum: this.currentPage,
          pageSize:this.pageSize,
          search: this.search
        }
      }).then(res =>{
        this.loading = false
        this.tableData = res.data.records
        this.total= res.data.total
      })
    },
    add(){
      this.dialogVisible=true
      this.form={}
      if (this.$refs['upload']) {
        this.$refs['upload'].clearFiles()
      }
    },
    save() {
      if (this.form.id) {
        request.put("/book", this.form).then(res => {
          console.log(res)
          if(res.code ==='0'){
            ElMessage.success("修改成功")
          } else {
            ElMessage.error("修改失败")
          }
          this.load()
          this.dialogVisible = false
        })
      } else {
        request.post("/book", this.form).then(res => {
          console.log(res)
          if(res.code ==='0'){
            ElMessage.success("新增成功")
          } else {
            ElMessage.error("新增失败")
          }
          this.load()
          this.dialogVisible = false
        })
      }
    },
    handleEdit(row){
      this.form = JSON.parse(JSON.stringify(row))
      this.dialogVisible = true
      this.$nextTick(() => {
        if (this.$refs['upload']) {
          this.$refs['upload'].clearFiles()  // 清除历史文件列表
        }
      })
    },
    handleDelete(id){
      console.log(id)
      request.delete("/book/" + id).then(res=>{
        if(res.code ==='0'){
          ElMessage.success("删除成功")
        } else {
          ElMessage.error("删除失败")
        }
        this.load()
      })
    },
    handleSizeChange(pageSize){
      this.pageSize= pageSize
      this.load()
    },
    handleCurrentChange(pageNum){
      this.currentPage = pageNum
      this.load()
    },
  }
}
</script>
