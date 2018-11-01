<template>
   <div style="padding:20px;" id="app">
        <div class="panel panel-primary">
            <table class="table table-bordered table-striped text-center">
                <thead>
                    <tr>
                        <th>序号</th>
                        <th>用户名</th>
                        <th>性别</th>
                        <th>生日</th>
                        <th>操作</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for ="(user,index) in users">
                      <td>{{index+1}}</td>
                      <td>{{user.studentName}}</td>
                      <td>{{user.studentSex}}</td>
                      <td>{{user.stuBirthday}}</td>
                      <td><el-button type="danger" icon="el-icon-delete" circle v-on:click="remove(index)"></el-button></td>
                      <td><el-button type="primary" icon="el-icon-edit" circle style="margin-left:-200px" @click="openEditDialog = true;setData(index)"></el-button></td>
                    </tr>
                    <tr>
                      <td></td>
                      <td><el-input placeHolder='用户名' type="text"  id="studentName" v-model="user.studentName"/></td>
                      <td><el-input placeHolder='性别' type="text" id="studentSex"v-model="user.studentSex"/></td>
                      <td><el-input placeHolder='生日' type="text" id="stuBirthday"v-model="user.stuBirthday"/></td>
                      <td><el-button type="primary" @click="insert">增加</el-button></td>
                    </tr>
                </tbody>
            </table>
        </div>
        <div style="float:left;margin-left:100px;margin-top:30px;margin-bottom:30px">
          <el-input style="width:500px;" placeHolder='请输入用户名' id="search" type='text' v-model='keyword'/>
          <el-button style=""type='primary' @click="search">搜索</el-button>
          <p style="margin-top:30px">查询结果</p>
          <table class="table table-bordered table-striped text-center">
              <thead>
                  <tr>
                      <th>序号</th>
                      <th>用户名</th>
                      <th>性别</th>
                      <th>生日</th>
                  </tr>
              </thead>
              <tbody>
                  <tr v-for ="(user,index) in searchRes">
                    <td>{{index+1}}</td>
                    <td>{{user.studentName}}</td>
                    <td>{{user.studentSex}}</td>
                    <td>{{user.stuBirthday}}</td>
                  </tr>
              </tbody>
          </table>
        </div>
        <el-dialog title="修改信息" :visible.sync="openEditDialog">
          <table>
              <thead>
                  <tr>
                      <th>用户名</th>
                      <th>性别</th>
                      <th>生日</th>
                  </tr>
              </thead>
              <tbody>
                  <tr>
                    <td><el-input placeHolder='用户名' type="text"  id="studentName" v-model="this.users[editIndex].studentName"/></td>
                    <td><el-input placeHolder='性别' type="text" id="studentSex" v-model="this.users[editIndex].studentSex"/></td>
                    <td><el-input placeHolder='生日' type="text" id="stuBirthday" v-model="this.users[editIndex].stuBirthday"/></td>
                  </tr>
              </tbody>
          </table>
          <div slot="footer" class="dialog-footer">
             <el-button @click="openEditDialog = false">取 消</el-button>
             <el-button type="primary" @click="openEditDialog = false;commit()">确 定</el-button>
          </div>
        </el-dialog>
    </div>
</template>

<script>
export default {
  name: 'hello',
  data () {
    return {
      user: {'studentID':'','studentName': '', 'studentSex': '', 'stuBirthday': ''},
      users: [],
      keyword:'',
      searchRes:[],
      editIndex:0,
      openEditDialog: false
    }
  },
  mounted() {
    this.requestInitData();
  },
  methods: {
    requestInitData() {
      this.$axios.get('/spring/getAllStudents').then((response)=>{
         console.log(response);
         this.$message({
          message: '数据初始化成功',
          type: 'success'
        });
         this.users = response.data.data;
       }).catch((error)=>{
         console.log(error);
         //初始化失败
         this.$message.error('数据初始化失败');
       })
    },
    insert: function () {
      //增加一条数据
      this.$axios.get("/spring/insertStudent?studentName=" + this.user.studentName +
        "&studentSex=" + this.user.studentSex + "&stuBirthday=" + this.user.stuBirthday
        ).then((response)=>{
          if (response.data.flag === 200) {
            this.users.push(this.user)
            this.$message({
             message: '数据添加成功',
             type: 'success'
           });
          } else {
            this.$message.error('数据添加失败');
          }
        }).catch((error)=>{
            this.$message.error('数据添加失败');
        })
    },
    remove: function (index) {
      this.$confirm('此操作将永久删除该条数据, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$axios.get("/spring/deleteStudent?studentID=" + this.users[index].studentID).then((response)=>{
            this.$message({
               message: '数据删除成功',
               type: 'success'
             });
             this.users.splice(index, 1);
          }).catch((error)=>{
            this.$message.error('数据删除失败');
          })
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });
        });
    },
    search: function () {
      this.$axios.get("/spring/getStudentWithName?studentName=" + this.keyword).then((response)=>{
        console.log(response);
        if (response.data.flag === 200) {
          this.searchRes = response.data.data;
          //查询成功
          this.$message({
             message: '查询成功',
             type: 'success'
           });
        } else {
          //未找到结果
          this.$message.error('未找到匹配结果');
        }
      }).catch((error)=>{
        this.$message.error('数据查询失败');
      })
    },
    setData: function(index) {
      //打开编辑菜单
      this.editIndex = index;
    },
    commit: function() {
      this.$axios.get("/spring/updateStudent?studentID=" + this.users[this.editIndex].studentID
      + "&studentName="+ this.users[this.editIndex].studentName + "&studentSex=" + this.users[this.editIndex].studentSex+
      "&stuBirthday=" + this.users[this.editIndex].stuBirthday
    ).then((response)=>{
      if (response.data.flag === 200) {
        this.$message({
           message: '数据更新成功',
           type: 'success'
         });
      } else {
        this.$message.error('数据更新失败');
      }
      }).catch((error)=>{
        this.$message.error('数据更新失败');
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
tr,th{
  width: 300px;
  text-align:center;
}
</style>
