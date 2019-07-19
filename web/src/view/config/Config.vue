<template>
    <div>
        <el-card shadow="never">
            <el-row class="app-btn-group">
                <el-col :span="4">
                    <el-button v-if="$root.CheckPriv($root.Priv.PROJECT_NEW)" @click="openAddDialogHandler" icon="iconfont left small icon-add" size="medium" type="primary">{{ $t('add_config') }}</el-button>&nbsp;
                </el-col>
                <el-col :span="6" :offset="14">
                    <el-input @keyup.enter.native="searchHandler" v-model="searchInput" size="medium" :placeholder="$t('please_input_keyword')">
                        <el-button @click="searchHandler" slot="append" icon="el-icon-search"></el-button>
                    </el-input>
                </el-col>
            </el-row>
            
            <el-table
                class="app-table"
                size="medium"
                v-loading="tableLoading"
                :data="tableData">
                <el-table-column prop="name" :label="$t('config_name')"></el-table-column>

                <el-table-column :label="$t('operate')" width="380" align="right">
                    <template slot-scope="scope">
                        <el-button
                        v-if="$root.CheckPriv($root.Priv.PROJECT_VIEW)"
                        icon="el-icon-view"
                        type="text"
                        @click="openViewDialogHandler(scope.row)">{{ $t('view') }}</el-button>
                        <el-button
                        v-if="$root.CheckPriv($root.Priv.PROJECT_EDIT)"
                        icon="el-icon-edit"
                        type="text"
                        @click="openEditDialogHandler(scope.row)">{{ $t('edit') }}</el-button>
                        <el-button
                        v-if="$root.CheckPriv($root.Priv.PROJECT_DEL)"
                        type="text"
                        icon="el-icon-delete"
                        class="app-danger"
                        @click="deleteHandler(scope.row)">{{ $t('delete') }}</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <el-pagination
                background
                layout="prev, pager, next"
                class="app-pagination"
                @current-change="currentChangeHandler"
                :current-page.sync="$root.Page"
                :page-size="$root.PageSize"
                :total="$root.Total">
            </el-pagination>
        </el-card>

        <el-dialog :top="$root.DialogNormalTop" :width="$root.DialogLargeWidth" :title="dialogTitle" :visible.sync="dialogVisible" @close="dialogCloseHandler">
            <div class="app-dialog" v-loading="dialogLoading">
                <el-form class="app-form" ref="dialogRef" :model="dialogForm" size="medium" label-width="130px">
                    <h4 class="app-form-subtitle">{{ $t('base_information') }}</h4>
                    <el-form-item 
                    :label="$t('config_name')"
                    prop="name"
                    :rules="[
                        { required: true, message: $t('name_cannot_empty'), trigger: 'blur'},
                    ]">
                        <el-input :placeholder="$t('please_input_project_name')" v-model="dialogForm.name" autocomplete="off"></el-input>
                    </el-form-item>

                    <el-form-item
                    :label="$t('description')"
                    prop="description">
                        <el-input :placeholder="$t('please_input_project_description')" type="textarea" :rows="3" v-model="dialogForm.description" autocomplete="off"></el-input>
                    </el-form-item>

                    <div class="app-divider"></div>
                    <h4 class="app-form-subtitle">{{ $t('config_part_1') }}</h4>

                    <el-form-item
                    :label="$t('param_1')"
                    prop="param_1"
                    :rules="[
                        { required: true, message: $t('cannot_empty'), trigger: 'blur'},
                    ]">
                        <el-input :placeholder="$t('param_1')" v-model="dialogForm.param_1" autocomplete="off"></el-input>
                    </el-form-item>

                    <el-form-item
                    :label="$t('param_2')"
                    prop="param_2"
                    :rules="[
                        { required: true, message: $t('cannot_empty'), trigger: 'blur'},
                    ]">
                        <el-radio-group v-model="dialogForm.param_2">
                            <el-radio :label="1">{{ $t('option_1') }}</el-radio>
                            <el-radio :label="2">{{ $t('option_2') }}</el-radio>
                        </el-radio-group>
                    </el-form-item>
                    <el-form-item 
                    :label="$t('param_3')"
                    prop="param_3"
                    :rules="[
                        { required: true, message: $t('cannot_empty'), trigger: 'blur' },
                    ]">
                        <el-select
                        class="app-input-mini"
                        v-model="dialogForm.param_3"
                        filterable
                        clearable
                        :placeholder="$t('please_select')">
                            <el-option
                            v-for="item in options"
                            :key="item.value"
                            :label="item.lable"
                            :value="item.value">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item
                    :label="$t('upload_file')">
                        <el-upload
                            class="upload-demo"
                            action="https://jsonplaceholder.typicode.com/posts/"
                            :on-preview="handlePreview"
                            :on-remove="handleRemove"
                            :before-remove="beforeRemove"
                            multiple
                            :limit="3"
                            :on-exceed="handleExceed"
                            :file-list="fileList">
                        <el-button size="small" type="primary">点击上传</el-button>
                        </el-upload>
                    </el-form-item>

                </el-form>
                <div slot="footer" class="dialog-footer">
                    <el-button size="small" @click="dialogCloseHandler">{{ $t('cancel') }}</el-button>
                    <el-button :loading="btnLoading" size="small" type="primary" @click="dialogSubmitHandler">{{ $t('enter') }}</el-button>
                </div>
            </div>
        </el-dialog>

        <el-dialog :top="$root.DialogNormalTop" :width="$root.DialogLargeWidth" :title="$t('view_config_info')" :visible.sync="dialogViewVisible" @close="dialogViewVisible = false">
            <div class="app-dialog" v-loading="dialogViewLoading">
                <el-form size="medium" label-width="130px">
                    <h4 class="app-form-subtitle">{{ $t('base_information') }}</h4>
                    <el-form-item 
                    :label="$t('config_id')">
                        {{ dialogViewForm.id }}
                    </el-form-item>

                    <el-form-item  :label="$t('config_name')">
                        {{ dialogViewForm.name }}
                    </el-form-item>

                    <el-form-item :label="$t('description')">
                        {{ dialogViewForm.description }}
                    </el-form-item>

                    <div class="app-divider"></div>
                    <h4 class="app-form-subtitle">{{ $t('config_part_1') }}</h4>

                    <el-form-item :label="$t('param_1')">
                        {{ dialogViewForm.param_1 }}
                    </el-form-item>

                    <el-form-item :label="$t('param_2')">
                        {{ dialogViewForm.param_2 }}
                    </el-form-item>
                
                    <el-form-item :label="$t('param_3')">
                        {{ dialogViewForm.param_3 }}
                    </el-form-item>

                </el-form>
            </div>
        </el-dialog>

    </div>
</template>

<script>
import { 
    listSpaceApi, 
    detailSpaceApi, 
    newProjectApi, 
    updateProjectApi, 
    listProjectApi, 
    switchStatusProjectApi, 
    detailProjectApi, 
    deleteProjectApi, 
    updateBuildScriptApi
} from '@/api/project'
import { listGroupApi } from '@/api/server' 
import codeMirror from 'codemirror/lib/codemirror.js'
import 'codemirror/lib/codemirror.css'
import 'codemirror/mode/shell/shell.js'
import 'codemirror/theme/dracula.css'
import 'codemirror/addon/scroll/simplescrollbars.css'
import 'codemirror/addon/scroll/simplescrollbars.js'

export default {
    data() {
        return {
            editorInstance: null,
            dialogBuildVisible: false,
            dialogBuildLoading: false,
            dialogBuildForm: {
                id: 0,
                build_script: '',
            },

            searchInput: '',
            tableLoading: false,
            tableData: [],

            dialogViewVisible: false,
            dialogViewLoading: false,
            dialogViewForm: {},

            dialogVisible: false,
            dialogTitle: '',
            dialogLoading: false,
            dialogForm: {
                id: 0,
                name: '',
                description: '',
                param_1: '',
                param_2: '',
                param_3: '',
            },
            btnLoading: false,

            dialogSpaceVisible: false,
            spaceId: undefined,
            spaceLoading: false,
            spaceDetail: {},
            spaceList: [],

            // clusterList: [],
            // selectOnlineCluster: undefined,
            options: [{
                value: '选项1',
                label: '选项1'
            }, {
                value: '选项2',
                label: '选项2'
            }, {
                value: '选项3',
                label: '选项3'
            }],
        }
    },
    watch: {
        spaceId() {
            if (this.spaceId) {
                this.spaceLoading = true
                detailSpaceApi({id: this.spaceId}).then(res => {
                    this.spaceDetail = res
                    this.spaceLoading = false
                }).catch(err => {
                    this.spaceLoading = false
                })
                this.$root.PageInit()
                this.loadTableData()
            }
        }
    },
    methods: {
        openViewDialogHandler(row) {
            this.dialogViewVisible = true
            this.dialogViewLoading = true
            detailProjectApi({id: row.id}).then(res => {
                this.dialogViewForm = res
                this.dialogViewLoading = false
            })
        },
        deleteHandler(row) {
            this.$root.ConfirmDelete(() => {
                deleteProjectApi({id: row.id}).then(res => {
                    this.$root.MessageSuccess()
                    this.$root.PageReset()
                    this.loadTableData()
                })
            })
        },
        enableSwitchHandler(val, row) {
            switchStatusProjectApi({status: val, id: row.id}).then(res => {
            }).catch(err => {
                row.status = Number(!val)
            })
        },
        searchHandler() {
            this.$root.PageInit()
            this.loadTableData()
        },
        openAddDialogHandler() {
            this.dialogVisible = true
            this.dialogTitle = this.$t('add_project')
        },
        openEditDialogHandler(row) {
            this.dialogVisible = true
            this.dialogTitle = this.$t('edit_project')
            this.dialogLoading = true
            detailProjectApi({id: row.id}).then(res => {
                this.dialogLoading = false
                this.dialogForm = res
            }).catch(err => {
                this.dialogCloseHandler()
            })
        },
        dialogSubmitHandler() {
            let vm = this
            this.$refs.dialogRef.validate((valid) => {
                if (!valid) {
                    return false;
                }
                this.btnLoading = true
                let opFn
                if (this.dialogForm.id) {
                    opFn = updateProjectApi
                } else {
                    opFn = newProjectApi
                }
                this.dialogForm.space_id = this.spaceId
                opFn(this.dialogForm).then(res => {
                    this.$root.MessageSuccess(() => {
                        this.dialogCloseHandler()
                        this.btnLoading = false
                        this.loadTableData()
                    })
                }).catch(err => {
                    this.btnLoading = false
                })
            });
        },
        dialogCloseHandler() {
            this.dialogVisible = false
            this.dialogLoading = false
            this.btnLoading = false
            this.$refs.dialogRef.resetFields();
            this.dialogForm = {}
        },
        // selectClusterHandler(clusterId) {
        //     this.selectOnlineCluster = undefined
        //     if (!clusterId) {
        //         return
        //     }
        //     if (!this.dialogForm.online_cluster) {
        //         this.dialogForm.online_cluster = []
        //     }
        //     if (this.dialogForm.online_cluster.indexOf(clusterId) == -1) {
        //         this.dialogForm.online_cluster.push(clusterId)
        //     }
        // },
        // removeClusterHandler(clusterId) {
        //     if (this.dialogForm.online_cluster) {
        //         this.dialogForm.online_cluster.forEach((id, index) => {
        //             if (id == clusterId) {
        //                 this.dialogForm.online_cluster.splice(index, 1)
        //             }
        //         })
        //     }
        // },
        selectSpaceHandler() {
            this.dialogSpaceVisible = false
        },
        switchSpaceHandler() {
            this.dialogSpaceVisible = true
        },
        // formatClusterName(id) {
        //     let name = '--'
        //     this.clusterList.forEach(cluster => {
        //         if (id == cluster.id) {
        //             name = cluster.name
        //         }
        //     })
        //     return name
        // },
        currentChangeHandler() {
            this.loadTableData()
        },
        loadTableData() {
            this.tableLoading = true
            listProjectApi({space_id: this.spaceId, keyword: this.searchInput, offset: this.$root.PageOffset(), limit: this.$root.PageSize}).then(res => {
                this.tableData = res.list
                this.$root.Total = res.total
                this.tableLoading = false
            }).catch(err => {
                this.tableLoading = false
            })
        },
        loadSpaceList() {
            listSpaceApi({offset: 0, limit: 999}).then(res => {
                if (res.list) {
                    this.spaceList = res.list
                }
                this.initSpaceId()
            })
        },
        // loadClusterList() {
        //     listGroupApi({offset: 0, limit: 999}).then(res => {
        //         if (res.list) {
        //             this.clusterList = res.list
        //         }
        //     })
        // },
        initSpaceId() {
            if (this.spaceList.length && !this.spaceId) {
                this.spaceId = this.spaceList[0].id
            }
        },
    },
    mounted() {
        this.loadSpaceList()
        // this.loadClusterList()
    }
}
</script>
