<template>
  <div>
    <el-row type="flex">
      <el-col :span="4">
        <div class="grid-content bg-purple">
          <el-menu default-active="2" class="el-menu-demo">
            <el-menu-item v-bind="{index:'index'+index}" v-for="(item,index) in lists" :key="index" @click="switchInfo(item.id)">{{item.name}}</el-menu-item>
          </el-menu>
          <el-button @click="dialogFormVisible = true" class="addAppDataButton">添加APP</el-button>
          <!-- Form -->
          <el-dialog title="添加APP" :visible.sync="dialogFormVisible">
            <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
              <el-form-item label="名称" prop="name">
                <el-input v-model="ruleForm.name"></el-input>
              </el-form-item>
              <el-form-item label="描述" prop="description">
                <el-input v-model="ruleForm.description"></el-input>
              </el-form-item>
              <el-form-item label="备注">
                <el-input v-model="ruleForm.remark"></el-input>
              </el-form-item>
              <el-form-item>
                <el-button @click="resetAppForm('ruleForm')">重置</el-button>
                <el-button type="primary" @click="submitAppForm('ruleForm')">提交</el-button>
              </el-form-item>
            </el-form>
          </el-dialog>
        </div>
      </el-col>
      <el-col :span="20" class="mainContent">
        <div class="grid-content bg-purple-light">
          <el-alert v-if="lists.length ===0" title="内容为空，请点击左侧的添加APP按钮" type="info">
          </el-alert>
          <div class="app-detail">
            <div class="app-info">
              <div class="app-name">
                <div class="app-name-header">{{detailInfo.name}}</div>
                <el-button-group>
                  <el-button icon="edit" size="small" @click="dialogFormVisible = true">编辑</el-button>
                  <el-button icon="delete" size="small" type="danger" @click="deleteAppData(detailInfo.id)">删除</el-button>
                </el-button-group>
                <!-- Form -->
                <el-dialog :title="appTitle" :visible.sync="dialogFormVisible">
                  <el-form :model="detailInfo" :rules="rules" ref="detailInfo" label-width="100px" class="demo-ruleForm">
                    <el-form-item label="名称" prop="name">
                      <el-input v-model="detailInfo.name"></el-input>
                    </el-form-item>
                    <el-form-item label="描述" prop="description">
                      <el-input v-model="detailInfo.description"></el-input>
                    </el-form-item>
                    <el-form-item label="备注">
                      <el-input v-model="detailInfo.remark"></el-input>
                    </el-form-item>
                    <el-form-item>
                      <el-button @click="resetAppForm('detailInfo')">重置</el-button>
                      <el-button type="primary" @click="submitAppForm('detailInfo',detailInfo.id)">提交</el-button>
                    </el-form-item>
                  </el-form>
                </el-dialog>
              </div>
              <div class="app-desc">
                <div class="app-name-header">APP 描述</div>
                <div class="app-remark-detail">{{detailInfo.description}}</div>
              </div>
              <div class="app-remark" v-if="detailInfo.remark !== ''">
                <div class="app-name-header">备注信息</div>
                <div class="app-remark-detail">{{detailInfo.remark}}</div>
              </div>
              <div class="app-created">
                <div class="app-name-header">创建时间</div>
                <div class="app-remark-detail">{{detailInfo.created_at}}</div>
              </div>
            </div>
            <div class="divider"></div>
            <div class="app-env">
              <div>
                <div class="app-env-header">
                  <environment-info :appId="currentAppID" ref="environmentDom"></environment-info>
                </div>
              </div>
            </div>
            <div class="divider"></div>
            <div class="app-category">
              <div class="app-category-header">
                <category-info :appId="currentAppID" :cateId="cateId" ref="categoryDom" @refreshCate="refreshCategory"></category-info>
              </div>
              <el-row>
                <el-col :span="10">
                  <el-tree :data="categoryMenuData" :props="defaultProps" node-key="id" default-expand-all :expand-on-click-node="false" :render-content="renderContent" >
                  </el-tree>
                </el-col>
              </el-row>
            </div>
          </div>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
var id = 1000
import appCategory from './category.vue';
import environment from './environment.vue';
export default {
  components: {
    'category-info': appCategory,
    'environment-info':environment
  },
  data () {
    return {
      appTitle: '添加APP',
      lists: [],
      isInputEmpty: false,
      dialogFormVisible: false,
      currentAppID: 0,
      currentEnvId: 0,
      cateId:0,
      envTableData:[],
      ruleForm: {
        name: '',
        description: '',
        remark: ''
      },
      // app信息表单验证
      rules: {
        name: [
          { required: true, message: '请输入名称', trigger: 'blur' },
          { min: 1, max: 50, message: '长度在 1 到 50 个字符', trigger: 'blur' }
        ],
        description: [
          { required: true, message: '请输入描述信息', trigger: 'blur' },
          { min: 1, max: 200, message: '长度在 1 到 200 个字符', trigger: 'blur' }
        ]
      },
      detailInfo: {},
      categoryMenuData: [],
      defaultProps: {
        children: 'children',
        label: 'label'
      }
    }
  },
  created () {
    this.getAppLists()
  },

  methods: {
    // 获取APP导航菜单信息
    getAppLists () {
      var url = this.$common.baseUrl + '/apps'
      this.$http.get(url).then(function (res) {
        if (res.status !== 200) {
          this.$common.errorMsg(res.status)
          return false
        }
        var data = res.body || []
        this.httpStatus = res.status
        this.lists = data
        if (data[0]) {
          this.switchInfo(data[0].id)
        }
      })
    },
    // 提交APP表单
    submitAppForm (formName,id) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          this.dialogFormVisible = false
          if(this.appTitle == '添加APP'){
            var url = this.$common.baseUrl +  '/apps'
            this.$http.post(url, { name: this.ruleForm.name, description: this.ruleForm.description, remark: this.ruleForm.remark }, { emulateJSON: true }).then(function (res) {
              if (res.status === 200 && res.body.id) {
                this.$common.successMsg('APP添加成功')
                this.getAppLists()
              } else {
                this.$common.errorMsg(res.status)
              }
            })
            
          } else {
              var url = this.$common.baseUrl + '/apps/' + id
              this.$http.put(url, { name: this.detailInfo.name, description: this.detailInfo.description, remark: this.detailInfo.remark }, { emulateJSON: true }).then(function (res) {
                if (res.status === 200 && res.body.id) {
                  this.$common.successMsg('修改APP信息成功')
                  this.getAppLists()
                  return true
                }
                this.$common.errorMsg(res.status)
              })
          }
          this.resetAppForm()
        } else {
          this.$common.errorMsg('提交APP信息失败')
        }
      })
    },
    // 重置APP表单
    resetAppForm (formName) {
       this.$refs[formName].resetFields();
    },
    // 切换APP导航菜单
    switchInfo (id) {
      this.currentAppID = id
      var url = this.$common.baseUrl + '/apps/' + id
      this.$http.get(url).then(function (res) {
        if (res.status === 200) {
          this.detailInfo = res.body
          // this.getEnvData(id)
          this.$refs.environmentDom.getEnvData(id);
          this.getCategoryMenuData(id)
        } else {
          this.$common.errorMsg(res.status)
        }
      })
    },
    // 删除APP信息
    deleteAppData (id) {
      var url = this.$common.baseUrl + '/apps/' + id
      this.$http.delete(url).then(function (res) {
        if (res.status === 200 && res.body.id) {
          this.$common.successMsg('删除APP数据成功')
          this.getAppLists()
          return false
        }
        this.$common.errorMsg(res.status)
      })
    },
    //  获取环境配置信息
    getEnvData (id) {
      var url = this.$common.baseUrl + '/envs/app/' + id;
      this.$http.get(url).then(function (res) {
        if (res.status === 200) {
          this.envTableData = res.body
        } else {
          this.$common.errorMsg(res.status)
        }
      }, function (res) {
        this.envTableData = []
        this.$common.errorMsg(res.status)
      })
    },
    refreshCategory(){
      this.getCategoryMenuData(this.currentAppID);
    },
    // 获取树形菜单信息
    getCategoryMenuData (appId) {
      var url = this.$common.baseUrl + '/categories/app/' + appId + '?type=tree'
      this.$http.get(url).then(function (res) {
        if (res.status !== 200) {
          this.$common.errorMsg(res.status)
          this.categoryMenuData = []
          return false
        }
        console.log(res.body, new Date())
        this.categoryMenuData = res.body
      }, function (res) {
        this.$common.errorMsg(res.status)
        this.categoryMenuData = []
      })
    },
    // 编辑树形菜单节点
    editCategory (data) {
      this.cateId = data.id;
      this.$refs.categoryDom.getCategoryForm(data.id);
    },
    // 删除树形菜单节点
    remove ( data) {
      // store.remove(data.id);
      var url = this.$common.baseUrl + '/categories/' + data.id;
      this.$http.delete(url).then(function (res) {
        if(res.status !== 200){
          this.$common.errorMsg(res.status);
          return false;
        }
        this.$common.successMsg('删除分类信息成功');
        this.refreshCategory();
      })
    },
    // 渲染树形菜单
    renderContent (h, { node, data, store }) {
      return (
        <span>
          <span>
            <span>{node.label}</span>
          </span>
          <span style="float: right; margin-right: 20px">
            <el-button-group>
              <el-button size="small" plain icon="edit" type="info" on-click={() => this.editCategory( data)}>edit </el-button>
              <el-button size="small" icon="delete" type="danger" on-click={() => this.remove(data)}> Delete </el-button>
            </el-button-group>
          </span>
        </span>)
    }
  }
}
</script>
<style scoped>
.addAppDataButton {
  width: 100%;
}

.app-desc,
.app-created {
  margin: 20px 0;
}

.app-remark-detail {
  margin: 5px 0;
}
.app-category-header{
  margin: 20px 0;
}
.app-env-header {
  margin-bottom: 20px;
}

</style>