<template>
<el-row>
  <el-col :span="24">
    <el-button-group>
      <el-button type="primary" v-if="groupManager_btn_add" icon="plus" @click="handlerAdd">添加</el-button>
      <el-button type="primary" v-if="groupManager_btn_edit" icon="edit" @click="handlerEdit">编辑</el-button>
      <el-button type="primary" v-if="groupManager_btn_del" icon="delete" @click="handleDelete">删除</el-button>
      <el-button type="success" v-if="groupManager_btn_resourceManager" @click="handlerAuthority">
        权限分配</el-button>
      <el-button type="success" v-if="groupManager_btn_userManager" @click="handlerUser">
        关联用户</el-button>
      <el-button type="warning" v-if="groupManager_btn_authorizeManager" @click="handlerAuthorize">
        权限下发</el-button>
    </el-button-group>
  </el-col>
  <el-col :span="8" style='margin-top:15px;'>
    <el-input placeholder="输入关键字进行过滤" v-model="filterText"> </el-input>
    <el-tree class="filter-tree" :data="treeData" node-key="id" highlight-current :props="defaultProps" :filter-node-method="filterNode"   :expand-on-click-node="false" ref="groupTree" @node-click="getNodeData" default-expand-all> </el-tree>
  </el-col>
  <el-col :span="16" style='margin-top:15px;'>
    <el-form :label-position="labelPosition" label-width="80px" :model="form" ref="form">
      <el-form-item label="名称">
        <el-input v-model="form.name" :disabled="formEdit"></el-input>
      </el-form-item>
      <el-form-item label="编码">
        <el-input v-model="form.code" :disabled="formEdit"></el-input>
      </el-form-item>
      <el-form-item label="描述">
        <el-input v-model="form.description" :disabled="formEdit"></el-input>
      </el-form-item>
      <el-form-item v-if="formStatus == 'update'">
        <el-button type="primary" v-if="groupManager_btn_edit" @click="update">更新</el-button>
        <el-button @click="onCancel">取消</el-button>
      </el-form-item>
      <el-form-item v-if="formStatus == 'create'">
        <el-button type="primary" v-if="groupManager_btn_add" @click="create">保存</el-button>
        <el-button @click="onCancel">取消</el-button>
      </el-form-item>
    </el-form>
  </el-col>
  <el-dialog :title="dialogUserName" :visible.sync="dialogUserVisible">
    <group-user :groupId="currentId" @closeUserDialog="closeUserDialog" ref="groupUser"></group-user>
  </el-dialog>
  <el-dialog :title="dialogAuthorityName" size="large" :visible.sync="dialogAuthorityVisible">
    <group-authority :groupId="currentId" @closeAuthorityDialog="closeAuthorityDialog" ref="groupAuthority"></group-authority>
  </el-dialog>
   <el-dialog :title="dialogAuthorizeName" size="large" :visible.sync="dialogAuthorizeVisible">
    <group-authorize :groupId="currentId" @closeAuthorizeDialog="closeAuthorizeDialog" ref="groupAuthorize"></group-authorize>
  </el-dialog>
</el-row>
</template>

<script>
import {
  fetchTree,
  getObj,
  addObj,
  delObj,
  putObj
} from '@/api/admin/group/index';
import { mapGetters } from 'vuex';
export default {
  name: 'groupDetail',
  components: {
    'group-user': () => import('./groupUser'),
    'group-authority': () => import('./groupAuthority'),
    'group-authorize': () => import('./groupAuthorize')
  },
  props: {
    type: {
      default: '1'
    }
  },
  data() {
    return {
      filterText: '',
      list: null,
      total: null,
      formEdit: true,
      formAdd: true,
      formStatus: '',
      dialogUserVisible: false,
      dialogUserName: '关联用户',
      dialogAuthorityVisible: false,
      dialogAuthorityName: '关联资源',
      dialogAuthorizeVisible: false,
      dialogAuthorizeName: '权限下发',
      listQuery: {
        groupType: this.type,
        name: undefined
      },
      treeData: [],
      defaultProps: {
        children: 'children',
        label: 'label'
      },
      labelPosition: 'right',
      groupManager_btn_edit: false,
      groupManager_btn_del: false,
      groupManager_btn_add: false,
      groupManager_btn_userManager: false,
      groupManager_btn_resourceManager: false,
      groupManager_btn_authorizeManager: false,
      form: {
        code: undefined,
        name: undefined,
        description: undefined,
        groupType: this.type
      },
      currentId: -1
    }
  },
  watch: {
    filterText(val) {
      this.$refs.groupTree.filter(val);
    }
  },
  created() {
    this.getList();
    this.groupManager_btn_edit = this.elements['groupManager:btn_edit'];
    this.groupManager_btn_del = this.elements['groupManager:btn_del'];
    this.groupManager_btn_add = this.elements['groupManager:btn_add'];
    this.groupManager_btn_userManager = this.elements['groupManager:btn_userManager'];
    this.groupManager_btn_resourceManager = this.elements['groupManager:btn_resourceManager'];
    this.groupManager_btn_authorizeManager = this.elements['groupManager:btn_authorizeManager'];
  },
  computed: {
    ...mapGetters([
      'elements'
    ])
  },
  methods: {
    getList() {
      fetchTree(this.listQuery).then(data => {
        this.treeData = data;
      });
    },
    filterNode(value, data) {
      if (!value) return true;
      return data.label.indexOf(value) !== -1;
    },
    getNodeData(data) {
      if (!this.formEdit) {
        this.formStatus = 'update';
      }
      getObj(data.id).then(response => {
        this.form = response.data;
      });
      this.currentId = data.id;
    },
    handlerEdit() {
      if (this.form.id) {
        this.formEdit = false;
        this.formStatus = 'update';
      }
    },
    handlerAdd() {
      this.resetForm();
      this.formEdit = false;
      this.formStatus = 'create';
    },
    handleDelete() {
      this.$confirm('此操作将永久删除, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        delObj(this.currentId).then(() => {
          this.getList();
          this.resetForm();
          this.onCancel();
          this.$notify({
            title: '成功',
            message: '删除成功',
            type: 'success',
            duration: 2000
          });
        });
      });
    },
    update() {
      putObj(this.form.id, this.form).then(() => {
        this.getList();
        this.$notify({
          title: '成功',
          message: '创建成功',
          type: 'success',
          duration: 2000
        });
      });
    },
    create() {
      addObj(this.form).then(() => {
        this.getList();
        this.$notify({
          title: '成功',
          message: '创建成功',
          type: 'success',
          duration: 2000
        });
      });
    },
    onCancel() {
      this.formEdit = true;
      this.formStatus = '';
    },
    resetForm() {
      this.form = {
        parentId: this.currentId,
        code: undefined,
        name: undefined,
        description: undefined,
        groupType: this.type
      };
    },
    handlerUser() {
      this.dialogUserVisible = true;
      if (this.$refs.groupUser !== undefined) {
        this.$refs.groupUser.groupId = this.currentId;
        this.$refs.groupUser.initUsers();
      }
    },
    handlerAuthority() {
      this.dialogAuthorityVisible = true;
      if (this.$refs.groupAuthority !== undefined) {
        this.$refs.groupAuthority.groupId = this.currentId;
        this.$refs.groupAuthority.initAuthoritys();
      }
    },
    handlerAuthorize() {
      this.dialogAuthorizeVisible = true;
      if (this.$refs.groupAuthorize !== undefined) {
        this.$refs.groupAuthorize.groupId = this.currentId;
        this.$refs.groupAuthorize.initAuthoritys();
      }
    },
    closeAuthorizeDialog() {
      this.dialogAuthorizeVisible = false;
    },
    closeUserDialog() {
      this.dialogUserVisible = false;
    },
    closeAuthorityDialog() {
      this.dialogAuthorityVisible = false;
    }
  }
}
</script>
