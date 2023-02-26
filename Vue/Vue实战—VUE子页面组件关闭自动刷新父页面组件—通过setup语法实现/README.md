### Vue实战—VUE子页面组件关闭自动刷新父页面组件—通过setup语法糖实现



#### **第一步、在父级页面创建自己页面的引用**

```vue
<template>
  <div class="m-user-table">
    <div class="footer">
      <div class="util">
        <el-button type="primary" @click="addHandler">
          <el-icon>
            <Plus/>
          </el-icon>
          新增用户
        </el-button>
      </div>
      <div class="table-inner">
        <el-table v-loading="loading" :data="userList" style="width: 100%; height: 100%" border>
          <el-table-column prop="code" label="用户Code" align="center" width="100"/>
          <el-table-column prop="nick_name" label="昵称" align="center" width="120"/>
          <el-table-column prop="role.name" label="角色" align="center" width="120">
            <template #default="scope">
              <el-tag v-for="(value,index) in scope.row.roles" :key="value.id" class="mx-1"
                      :class="index!==scope.row.roles.length-1?'m-tag-gap':''" size="default">
                {{ value.name }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="accounts" label="账号" align="center" width="120">
            <template #default="scope">
              <el-button type="primary" text @click="showAccount(scope.row.accounts)">
                查看
              </el-button>
            </template>
          </el-table-column>
          <el-table-column prop="integration" label="积分" align="center"/>
          <el-table-column prop="level" label="等级" align="center" width="120"/>
          <el-table-column prop="visit" label="文章访问量" align="center" width="120"/>
          <el-table-column prop="active" label="用户状态" align="center" width="120">
            <template #default="scope">
              <el-switch
                  inline-prompt
                  on-value="true"
                  off-value="false"
                  active-text="启用"
                  inactive-text="禁用"
                  v-model="scope.row.status"
                  @change="changeStatus(scope.row)"
              />
            </template>
          </el-table-column>
          <el-table-column prop="create_time" label="创建时间" align="center" width="180"/>
          <el-table-column prop="update_time" label="更新时间" align="center" width="180"/>
          <el-table-column prop="operator" label="操作" width="200px" align="center" fixed="right">
            <template #default="scope">
              <el-button type="primary" size="small" icon="Edit" @click="editHandler(scope.row)">
                编辑
              </el-button>
              <el-button @click="del(scope.row)" type="danger" size="small" icon="Delete">
                删除
              </el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div class="pagination">
        <el-pagination
            :currentPage="currentPage"
            :page-size="userListRequestParam.limit"
            background
            layout="total, sizes, prev, pager, next, jumper"
            :total="total"
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
        />
      </div>
    </div>

    //@refresh 这个命名可以自己改，UserDialog是你子页面名字  ，loadUserList是我的方法名，ref就是我调用子页面弹窗的名字
    <UserDialog @refresh="loadUserList" ref="userDialog"/>
  </div>
</template>
// 使用setup语法
<script lang="ts" setup>
import {ElMessageBox, ElMessage, FormInstance} from 'element-plus'
import {Search} from '@element-plus/icons-vue'
import {onMounted, reactive, ref} from 'vue'
// 引用子组件
import UserDialog from './UserDialog.vue'
import {userClient} from '@/api'

// 控制弹窗显示 和 关闭
const dialogVisible = ref(false)
const userDialog = ref()
const ruleFormRef = ref<FormInstance>()
const formInline = reactive({})
const loading = ref(true)
const userList = ref([])
const currentPage = ref(1)
const total = ref(0)
const userListRequestParam = {
  ids: null,
  status: null,
  level: null,
  nick_name: null,
  sections: null,
  limit: 10,
  offset: 0,
}



const addHandler = () => {
  userDialog.value.show()
}
const editHandler = (row) => {
  userDialog.value.show(row)
}


/**
 * 加载列表数据
 */
const loadUserList = () => {
  loading.value = true
  userListRequestParam.sections = 'accounts,roles'
  userClient.list(userListRequestParam).then((resp) => {
    const list = resp.data
    total.value = resp.total
    for (let i = 0; i < list.length; i++) {
      list[i].status = list[i].active === 1
    }
    
    // userList 和表格绑定，userList数据更新，表格也是自动更新
    userList.value = list
    console.log(list)
  }).finally(() => {
    loading.value = false
  })
}

onMounted(() => {
  loadUserList()
})
</script>
```





#### 第二步、就简单了在子页面结束的时候使用父级的方法就可以了

```vue
<template>
  <el-dialog @close="close" v-model="dialogVisible" :title="title" width="50%">
    <el-form ref="ruleFormRef" :model="ruleForm" :rules="rules" label-width="100px">
      <el-form-item label="昵称" prop="nick_name">
        <el-input v-model="ruleForm.nick_name" placeholder="请输入昵称"/>
      </el-form-item>
      <el-form-item label="关联角色" prop="roles">
        <el-select
            v-model="ruleForm.roles"
            multiple
            placeholder="请选择"
            style="width: 100%"
        >
          <el-option
              v-for="item in roleList"
              :key="item.code"
              :label="item.name"
              :value="item.code"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="邮箱号" prop="email">
        <el-input v-model="ruleForm.email" placeholder="请输入邮箱号"/>
      </el-form-item>
      <el-form-item label="账户密码">
        <el-input
            v-model="ruleForm.password"
            placeholder='请输入账户密码,如果不输入默认 admin123456'
            type="password"
            clearable
            show-password
        />
      </el-form-item>
      <el-form-item label="用户状态" prop="status">
        <el-switch
            v-model="ruleForm.status"
            inline-prompt
            active-text="启用"
            inactive-text="禁用"
        ></el-switch>
      </el-form-item>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="handleSubmit(ruleFormRef)">确定</el-button>
      </span>
    </template>
  </el-dialog>
</template>
<script lang="ts" setup>
import {ElMessageBox, ElMessage, FormInstance, FormRules} from 'element-plus'
import {onMounted, reactive, ref} from 'vue'
import {roleClient, userClient} from "@/api";
import {validatorMethod, verifyEmail} from "@/utils/validate";

let roleList = ref([])
const ruleFormRef = ref<FormInstance>()
const dialogVisible = ref<boolean>(false)
const isEdit = ref<boolean>(false)
const title = ref('新增用户')

// 定义emits事件，这里refresh就是在父组件中监听的事件，父组件通过鉴定refresh时间调用load方法重新刷新表格数据
const emits = defineEmits<{
  (event: 'refresh'): void
}>();

const rules = reactive<FormRules>({
  nick_name: [
    {required: true, message: '请输入昵称', trigger: 'blur'},
    {min: 1, max: 20, message: '长度在 1 到 20 个字符', trigger: 'blur'},
  ],
  roles: [{required: true, message: '请选择角色', trigger: 'change'}],
  email: [
    {required: true, validator: validatorMethod(verifyEmail, '请输入正确的邮箱'), trigger: 'blur'}
  ],
})

const ruleForm = reactive({
  nick_name: null,
  code: null,
  roles: [],
  email: null,
  password: null,
  status: true,
})

function close() {
  ruleFormRef.value.resetFields()
  Object.keys(ruleForm).forEach((key) => {
    if (key === 'active') ruleForm[key] = true
    else if (key === 'roles') ruleForm[key] = []
    else ruleForm[key] = null
  })
}

// 根据item是否存在参数来决定是编辑还是新增  
const show = (item = {}) => {
  title.value = '新增用户'
  isEdit.value = false
  if (item['code'] !== undefined && item['code'] !== null) {
    title.value = '编辑用户'
    isEdit.value = true
    Object.keys(item).forEach((key) => {
      if (key === 'accounts') {
        const accounts = item['accounts']
        if (accounts !== undefined && accounts !== null) {
          const account = accounts.filter(function (account) {
            return account.identity_type === 1
          })
          ruleForm['email'] = account[0].identifier
          ruleForm['password'] = account[0].credential
        }
      } else if (key === 'roles') {
        const roles = item[key]
        roles.forEach(function (role) {
          ruleForm['roles'].push(role.code)
        })
      } else {
        ruleForm[key] = item[key]
      }
    })
  }
  dialogVisible.value = true
}

/**
 * 更新用户和账户信息
 */
const updateUserAccount = () => {
   userClient.update(ruleForm.code, {
    nick_name: ruleForm.nick_name,
    roles: ruleForm.roles,
    email: ruleForm.email,
    password: ruleForm.password,
    identity_type: 1,   //email 类型
    active: ruleForm.status === null ? null : (ruleForm.status ? 1 : 0),
  }).then(() => {
    ElMessage({
      message: '更新成功',
      type: 'success',
    })
  }).catch(() => {
    ElMessage({
      message: '更新失败',
      type: 'error',
    })
  })
}

/**
 * 创建User和账户
 */
const createUserAccount = () => {
  userClient.create({
    nick_name: ruleForm.nick_name,
    roles: ruleForm.roles,
    active: ruleForm.status ? 1 : 0,
    identity_type: 1,   //email 类型
    email: ruleForm.email,
    password: ruleForm.password != null ? ruleForm.password : 'admin123456'
  }).then(() => {
    ElMessage({
      message: '创建成功',
      type: 'success',
    })
  }).catch(() => {
    ElMessage({
      message: '创建失败',
      type: 'error',
    })
  })
}

/**
 * 处理提交事件
 * @param done
 */
const handleSubmit = async (done: () => void) => {
  await ruleFormRef.value.validate((valid, fields) => {
    if (valid) {
      if (isEdit.value) {
        //编辑用户信息
        updateUserAccount()
      } else {
        //创建新用户
        createUserAccount()
      }
      dialogVisible.value = false
    }
  })

  //触发refresh事件，通知父组件刷新数据
  emits('refresh')
}


// 子组件暴漏show方法
defineExpose({
  show,
})
</script>
```







#### 参考

* [VUE调用子页面弹窗或组件弹窗，关闭弹窗刷新父级页面主页面，通过this.$emit来实现](https://blog.csdn.net/weixin_45973327/article/details/120757980)
* [Vue3拒绝写return，用setup语法糖,让写Vue3更畅快](https://juejin.cn/post/7078865301856583717)