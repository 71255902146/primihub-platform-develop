<template>
  <div class="container">
    <div class="steps">
      <div class="step">
        <div class="step-icon">
          <div class="step-description"> 基础信息 </div>
        </div>
        <div class="inner-con">
          <el-form ref="form" :model="formData" :rules="rules" label-width="140px">
            <el-form-item label="任务名称" prop="taskName">
              <el-input v-model="formData.taskName" maxlength="32" show-word-limit placeholder="请输入任务名称,限32字" />
            </el-form-item>
            <el-form-item label="输出格式" prop="outputFormat">
              <el-select v-model="formData.outputFormat" placeholder="请选择">
                <el-option label="资源文件(csv)" :value="0" />
              </el-select>
            </el-form-item>
            <el-form-item label="求交结果名称" prop="resultName">
              <el-input v-model="formData.resultName" placeholder="请输入内容" />
            </el-form-item>
          </el-form>
        </div>
      </div>
      <div class="step">
        <div class="step-icon">
          <div class="step-description"> 数据引入 </div>
        </div>
        <div class="inner-con inner-con-second">
          <el-form ref="formData" :model="formData" :rules="rules" label-width="0px">
            <div class="flex organ-title">
              <div>
                <svg-icon icon-class="share" /> <i class="iconfont icon-plane" /> 发起方
              </div>
              <div>
                <svg-icon icon-class="association" /> 协作方
              </div>
            </div>
            <div class="header">
              <div class="organ-container">
                <div class="row-title">
                  <span>参与机构</span>
                </div>
                <div class="organ-container-right flex">
                  <div class="organ">
                    <el-select v-model="formData.ownOrganName" class="organ-select" style="display:inline-block;" disabled :placeholder="formData.ownOrganName" />
                  </div>
                  <div class="content-organ">{{ taskTypeText }}</div>
                  <div class="organ">
                    <el-select v-model="formData.otherOrganId" placeholder="请选择协作方">
                      <el-option
                        v-for="item in organList"
                        :key="item.globalId"
                        :label="item.globalName"
                        :value="item.globalId"
                      />
                    </el-select>
                  </div>
                </div>
                <el-popover
                  class="popover-container"
                  placement="top-start"
                  title="隐私求交步骤："
                  width="400"
                  trigger="hover"
                >
                  <div>
                    <p>(1)添加资源：发起方和协作方先在各自节点【资源管理】菜单下添加数据资源</p>
                    <p>(2)资源授权：协作方向发起方授权资源权限，设置资源公开或指定机构可见</p>
                    <p>(3)发起查询：发起方在本方【隐私求交】菜单下发起隐匿查询任务，并查看结果</p>
                  </div>
                  <div slot="reference">
                    <svg-icon icon-class="problem" />
                  </div>
                </el-popover>
              </div>
            </div>
            <div class="item-row">
              <div class="item flex">
                <div class="row-title">
                  <span>资源表</span>
                </div>
                <div class="row-right-container justify-content-between flex">
                  <el-form-item prop="ownResourceId">
                    <el-select v-model="formData.ownResourceName" placeholder="发起方资源表" clearable @focus="openDialog('own')" @clear="handleResourceClear('own')" />
                  </el-form-item>
                  <div class="right-container-center"><img :src="centerImg" alt="" width="24"></div>
                  <el-form-item prop="otherResourceId">
                    <el-select v-model="formData.otherResourceName" placeholder="协作方资源表" @focus="openDialog('other')" />
                  </el-form-item>
                </div>

              </div>
              <div class="item flex">
                <div class="row-title">
                  <span>关联键</span>
                </div>
                <div class="row-right-container justify-content-between flex">
                  <el-form-item prop="ownKeyword">
                    <el-select v-model="formData.ownKeyword" v-loading="selectLoading" multiple no-data-text="暂无数据" clearable placeholder="发起方关联键">
                      <el-option
                        v-for="(item,index) in ownOrganResourceField"
                        :key="index"
                        :label="item.fieldName"
                        :value="item.fieldName"
                      >
                        <template slot="default">
                          <div class="option-item">
                            <template v-if="item.fieldDesc">
                              <div class="option-label" :style="{'max-width': item.fieldDesc ? '200px' : '100%'}">
                                <el-tooltip class="item" effect="dark" :content="item.fieldName" placement="top-start">
                                  <span>{{ item.fieldName }}</span>
                                </el-tooltip>
                              </div>
                              <div class="option-desc">
                                <el-tooltip class="item" effect="dark" :content="item.fieldDesc" placement="top-start">
                                  <span>{{ item.fieldDesc ? item.fieldDesc.length>8 ? item.fieldDesc.slice(0,8)+'...' : item.fieldDesc : '' }}</span>
                                </el-tooltip>
                              </div>
                            </template>

                            <div v-else>
                              <span class="option-label">{{ item.fieldName }}</span>
                            </div>
                          </div>
                        </template>
                      </el-option>
                    </el-select>
                  </el-form-item>
                  <div class="right-container-center"><img :src="centerImg" alt="" width="24"></div>
                  <el-form-item prop="otherKeyword">
                    <el-select v-model="formData.otherKeyword" multiple no-data-text="暂无数据" placeholder="协作方关联键" clearable>
                      <el-option
                        v-for="(item,index) in otherOrganResourceField"
                        :key="index"
                        :label="item.fieldName"
                        :value="item.fieldName"
                      >
                        <template slot="default">
                          <div class="option-item">
                            <template v-if="item.fieldDesc">
                              <div class="option-label" :style="{'max-width': item.fieldDesc ? '200px' : '100%'}">
                                <el-tooltip class="item" effect="dark" :content="item.fieldName" placement="top-start">
                                  <span>{{ item.fieldName }}</span>
                                </el-tooltip>
                              </div>
                              <div class="option-desc">
                                <el-tooltip class="item" effect="dark" :content="item.fieldDesc" placement="top-start">
                                  <span>{{ item.fieldDesc ? item.fieldDesc.length>8 ? item.fieldDesc.slice(0,8)+'...' : item.fieldDesc : '' }}</span>
                                </el-tooltip>
                              </div>
                            </template>

                            <div v-else>
                              <span class="option-label">{{ item.fieldName }}</span>
                            </div>
                          </div>
                        </template>
                      </el-option>
                    </el-select>
                  </el-form-item>
                </div>
              </div>
            </div>
          </el-form>

        </div>
      </div>

      <div class="step">
        <div class="step-icon">
          <div class="step-description"> 高级配置 </div>
        </div>
        <div class="inner-con inner-con-high">
          <el-form ref="form" :model="formData" :rules="rules" label-width="140px">
            <el-row>
              <el-col :span="12">
                <el-form-item label="输出内容" prop="outputContent">
                  <el-select v-model="formData.outputContent" placeholder="请选择" @change="handleTaskTypeChange">
                    <el-option label="交集" :value="0" />
                    <el-option label="差集" :value="1" />
                  </el-select>
                </el-form-item>
              </el-col>
            </el-row>
            <el-row>
              <el-col :span="12">
                <el-form-item label="实现方法" prop="psiTag">
                  <el-select v-model="formData.psiTag" placeholder="请选择实现方式">
                    <el-option v-for="item in psiTagOptions" :key="item.value" :label="item.label" :value="item.value" />
                  </el-select>
                </el-form-item>
              </el-col>
            </el-row>
            <el-form-item v-if="formData.psiTag == 2" label="可信计算节点" prop="teeOrganId">
              <el-select v-model="formData.teeOrganId" placeholder="请选择可信计算节点" @click.native="selectTeeOrgan">
                <el-option v-for="item in teeOrganList" :key="item.globalId" :label="item.globalName" :value="item.globalId" />
              </el-select>
              <div v-if="!formData.otherOrganId" class="display-modal" @click="selectTeeOrgan" />
              <div class="form-tip-text">注：第三方可信计算节点为双方提供数据机密性和完整性保护的前提，并支持计算。</div>
            </el-form-item>
            <el-form-item label="备注" prop="remarks">
              <el-input
                v-model="formData.remarks"
                size="mini"
                type="textarea"
                resize="none"
                maxlength="200"
                minlength="3"
                show-word-limit
              />
            </el-form-item>
            <el-form-item>
              <el-button ref="btnRef" icon="el-icon-check" type="primary" :disabled="isRun" @click="handleSubmit">提交任务</el-button>
            </el-form-item>
          </el-form>
        </div>
      </div>
    </div>
    <el-dialog
      title="选择资源"
      :visible.sync="dialogVisible"
      top="10px"
      class="dialog"
      width="800px"
      :before-close="handleDialogCancel"
    >
      <div class="dialog-body">
        <div class="search-input">
          <el-input
            v-model="searchKeyword"
            placeholder="请输入内容"
            class="search"
            @change="handleSearchNameChange"
            @keyup.enter.native="searchResource"
          >
            <el-button slot="append" icon="el-icon-search" @click="searchResource" />
          </el-input>
        </div>
        <ResourceTableSingleSelect max-height="560" :data="resourceList" :show-status="false" :selected-data="selectResources && selectResources.resourceId" @change="handleResourceChange" />
      </div>
      <div class="dialog-footer flex align-items-center" :class="{'justify-content-between': pageCount>1,'justify-content-center': pageCount<=1}">
        <pagination v-if="pageCount>1" :limit.sync="pageSize" :page.sync="pageNo" :page-count="pageCount" :total="total" layout="total, prev, pager, next" @pagination="handlePagination" />
        <div>
          <el-button @click="handleDialogCancel">取 消</el-button>
          <el-button type="primary" @click="handleDialogSubmit">确 定</el-button>
        </div>
      </div>

    </el-dialog>
  </div>
</template>

<script>
import { getPsiResourceAllocationList, saveDataPsi } from '@/api/PSI'
import { getResourceList } from '@/api/fusionResource'
import { getAvailableOrganList } from '@/api/center'
import ResourceTableSingleSelect from '@/components/ResourceTableSingleSelect'
import Pagination from '@/components/Pagination'

const intersection = require('@/assets/intersection.svg')
const diffsection = require('@/assets/diffsection.svg')

export default {
  name: 'PSITask',
  components: {
    ResourceTableSingleSelect,
    Pagination
  },
  data() {
    return {
      role: '',
      resourceList: [],
      searchKeyword: '',
      selectResources: null,
      dialogVisible: false,
      allOrganList: [],
      organList: [],
      teeOrganList: [],
      taskTypeText: '交',
      resourceName: '',
      selectLoading: false,
      ownOrganResourceField: [],
      otherOrganResourceField: [],
      total: 0,
      currentPage: 1,
      pageNo: 1,
      pageSize: 5,
      pageCount: 0,
      isRun: false, // task running state
      isReset: false,
      taskId: 0,
      tableDataA: [],
      tableDataB: [],
      ownResourceList: [],
      otherResourceList: [],
      formData: {
        taskName: '',
        ownOrganId: 0,
        ownResourceId: '', // 本机构资源Id
        ownKeyword: [], // 本机构关联键
        otherOrganId: '',
        otherResourceId: '', // 其他机构资源Id
        otherKeyword: [], // 其他机构关联键
        outputFormat: 0, // 输出方式
        outputFilePathType: 0, // 输出路径
        outputContent: 0,
        resultOrgan: [],
        resultOrganIds: '',
        outputNoRepeat: 1, // 输出内容是否去重
        resultName: '',
        remarks: null,
        serverAddress: '',
        psiTag: 0
      },
      ownResourceName: '',
      otherResourceName: '',
      rules: {
        taskName: [
          { required: true, message: '请输入任务名称', trigger: 'blur' }
        ],
        ownResourceId: [
          { required: true, message: '请选择资源' }
        ],
        ownKeyword: [
          { required: true, message: '请选择关联键', trigger: 'blur' }
        ],
        otherResourceId: [
          { required: true, message: '请选择资源' }
        ],
        otherKeyword: [
          { required: true, message: '请选择关联键', trigger: 'blur' }
        ],
        outputFilePathType: [
          { required: true, message: '请选择输出资源路径', trigger: 'blur' }
        ],
        outputNoRepeat: [
          { required: true, message: '请选择是否去重' }
        ],
        resultName: [
          { required: true, message: '请输入结果名称' },
          { max: 50, message: '长度在50个字符以内', trigger: 'blur' }
        ],
        outputContent: [
          { required: true, message: '请选择输出内容' }
        ],
        outputFormat: [
          { required: true }
        ],
        resultOrgan: [
          { required: true, message: '请选择结果获取方' }
        ],
        psiTag: [
          { required: true, message: '请选择实现协议' }
        ]
      },
      timer: null,
      psiTagOptions: [
        {
          value: 0,
          label: 'ECDH'
        },
        {
          value: 1,
          label: 'KKRT'
        },
        {
          value: 2,
          label: 'TEE'
        }
      ]
    }
  },
  computed: {
    // 结果获取方
    resultOrgan() {
      const { ownOrganId, ownOrganName, otherOrganId, otherOrganName } = this.formData
      const options = [
        {
          organId: ownOrganId,
          organName: ownOrganName
        }
      ]
      if (otherOrganId) {
        options.push({
          organId: otherOrganId,
          organName: otherOrganName
        })
      }
      return options
    },
    centerImg() {
      return this.formData.outputContent === 0 ? intersection : this.formData.outputContent === 1 ? diffsection : intersection
    },
    resultName() {
      return `${this.formData.taskName}求交结果`
    }
  },
  watch: {
    resultName(newVal) {
      if (newVal) {
        this.formData.resultName = this.resultName
      } else {
        this.formData.resultName = ''
      }
    },
    'formData.psiTag'(newVal) {
      if (newVal === 2) {
        this.formData.outputNoRepeat = 1
      } else {
        this.formData.teeOrganId = ''
      }
    },
    'formData.otherOrganId'(newVal) {
      if (newVal) {
        this.teeOrganList = this.allOrganList.filter((item) => {
          return item.globalId !== newVal
        })
      } else {
        this.teeOrganList = this.allOrganList
      }
    },
    'formData.teeOrganId': {
      handler(newVal) {
        if (newVal) {
          this.organList = this.allOrganList.filter((item) => {
            return item.globalId !== newVal
          })
        } else {
          this.organList = this.allOrganList
        }
      },
      immediate: true
    }
  },
  async created() {
    this.formData.ownOrganId = this.$store.getters.userOrganId
    this.formData.ownOrganName = this.$store.getters.userOrganName
    this.formData.resultOrgan.push(this.formData.ownOrganId)
    await this.getAvailableOrganList()
  },
  methods: {
    handleDialogSubmit() {
      if (!this.selectResources) {
        this.$message({
          message: '请选择资源',
          type: 'warning'
        })
        return
      }
      console.log('selectResources', this.selectResources.resourceId, this.formData.otherResourceId)
      if (this.role === 'own') {
        this.ownOrganResourceField = this.selectResources ? this.selectResources.fieldList : []
        this.formData.ownKeyword = ''
        this.formData.ownResourceId = this.selectResources.resourceId
        this.formData.ownResourceName = this.selectResources.resourceName
      } else {
        this.otherOrganResourceField = this.selectResources ? this.selectResources.fieldList : []
        this.formData.otherKeyword = ''
        this.formData.otherResourceId = this.selectResources.resourceId
        this.formData.otherResourceName = this.selectResources.resourceName
      }
      this.dialogVisible = false
    },
    handleSearchNameChange(searchName) {
      console.log(searchName)
      // this.form.resourceName = searchName
    },
    searchResource() {
      this.pageNo = 1
      this.getResourceList()
    },
    openDialog(role) {
      this.searchKeyword = ''
      this.pageNo = 1
      this.role = role
      if (role === 'other' && this.formData.otherOrganId === '') {
        this.$message.error('请选择协作方')
        return
      }
      if (this.selectResources) {
        this.selectResources.resourceId = role === 'own' ? this.formData.ownResourceId : this.formData.otherResourceId
      }
      this.dialogVisible = true
      this.getResourceList()
    },
    async getResourceList() {
      const params = {
        pageNo: this.pageNo,
        pageSize: this.pageSize,
        organId: this.role === 'own' ? this.formData.ownOrganId : this.formData.otherOrganId,
        resourceName: this.searchKeyword
      }
      const { code, result } = await getResourceList(params)
      if (code === 0) {
        const { data, total, totalPage } = result
        this.total = total
        this.pageCount = totalPage
        this.resourceList = data
      }
    },
    handleDialogCancel() {
      this.dialogVisible = false
      this.searchKeyword = ''
      this.pageNo = 1
    },
    handleResourceChange(data) {
      this.selectResources = data
    },
    handlePagination(data) {
      this.pageNo = data.page
      this.getResourceList()
    },
    async getAvailableOrganList() {
      this.loading = true
      const res = await getAvailableOrganList()
      if (res.code === 0) {
        this.loading = false
        const { result } = res
        this.allOrganList = this.organList = result
      }
    },
    handleTaskTypeChange(value) {
      this.taskTypeText = value === 0 ? '交' : '差'
    },
    selectTeeOrgan() {
      if (!this.formData.otherOrganId) {
        this.$message.error('请先选择协作方再选可信第三方')
      }
    },
    async getPsiResourceAllocationList(params) {
      const { resourceName, organId } = params
      const res = await getPsiResourceAllocationList({
        resourceName,
        organId,
        pageSize: this.pageSize,
        serverAddress: this.formData.serverAddress
      })
      if (res.code === 0) {
        return res.result.data
      }
    },
    handleSubmit() {
      const enable = this.checkParams()
      if (!enable) return
      // max size is 200
      this.formData.resultName = this.formData.resultName.length > 200 ? this.formData.resultName.substring(0, 200) : this.formData.resultName
      this.$refs.form.validate(async valid => {
        if (valid) {
          this.formData.resultOrganIds = this.formData.resultOrgan.join(',')
          const ownKeyword = this.formData.ownKeyword.join(',')
          const otherKeyword = this.formData.otherKeyword.join(',')
          this.isRun = true
          const res = await saveDataPsi(Object.assign({}, this.formData, { ownKeyword, otherKeyword }))
          if (res.code === 0) {
            this.$message({
              message: '创建完成',
              type: 'success'
            })
            this.toResultPage(res.result.dataPsiTask.taskId)
          } else {
            this.isRun = false
          }
        } else {
          console.log('error submit!!')
          this.isRun = false
          return false
        }
      })
    },
    toResultPage(id) {
      this.$router.push({
        name: 'PSIDetail',
        params: { id }
      })
    },
    checkParams() {
      const { otherOrganId, otherKeyword, ownResourceId, otherResourceId, ownKeyword, taskName, resultName, teeOrganId, psiTag } = this.formData
      let message = ''
      let enable = true
      if (!taskName) {
        message = '请输入任务名称'
        enable = false
      } else if (!resultName) {
        message = '请输入结果名称'
        enable = false
      } else if (!otherOrganId) {
        message = '请选择求交机构'
        enable = false
      } else if (ownResourceId === '') {
        message = '请选择发起方资源'
        enable = false
      } else if (otherResourceId === '') {
        message = '请选择协作方资源'
        enable = false
      } else if (otherKeyword === '') {
        message = '请选择协作方关联键'
        enable = false
      } else if (ownKeyword === '') {
        message = '请选择发起方关联键'
        enable = false
      } else if (psiTag === 2 && !teeOrganId) {
        message = '请选择可信计算节点'
        enable = false
      }
      if (!enable) {
        this.$message({
          message,
          type: 'error'
        })
      }

      return enable
    },
    handleResourceClear(role) {
      if (role === 'own') {
        this.formData.ownResourceId = ''
        this.formData.ownResourceName = ''
        this.formData.ownKeyword = ''
        this.ownOrganResourceField = []
      } else if (role === 'other') {
        this.formData.otherResourceId = ''
        this.formData.otherResourceName = ''
        this.otherOrganResourceField = []
        this.otherKeyword = ''
      }
    }
  }
}
</script>
<style lang="scss" scoped>
@import "../../styles/variables.scss";
::v-deep .el-dialog__body{
  padding: 10px 20px;
}
.container{
  overflow: hidden;
  background: #fff;
  padding: 36px;
  border-radius: 8px;
  &::before{
    border-left: 1px solid #cccccc;
    content: '';
    display: block;
    height: 100%;
  }
}
.inner-con{
  background: #fff;
  margin: 32px 0;
  ::v-deep .el-form{
    width: 950px;
    margin: 0 auto;
  }
  .form-tip-text{
    font-size: 12px;
    color: #666;
  }
  .display-modal{
    position: absolute;
    height: 40px;
    width: 340px;
    top:0px;
    z-index: 1;
  }
}
.row-title {
  width: 140px;
  align-self: center;
  font-size: 14px;
  color: #333;
  width: 140px;
  flex-shrink: 0;
  text-align: right;
  padding-right: 12px;
  font-weight: 700;
  span {
    &:before {
      content: '*';
      color: #f56c6c;
      margin-right: 4px;
    }
  }
}
.inner-con-second{
  ::v-deep .el-select, ::v-deep .el-input{
    input{
      border:none;
    }
  }
  .organ-title{
    padding-left: 160px;
    width: 100%;
    margin: 32px 0 24px 0;
    div{
      flex:1;
      color: #333;
      font-size: 16px;
      font-weight: 700;
      &:last-child{
        padding-left: 70px;
      }
    }
  }
  .row-right-container{
    flex:1;
    border:1px solid #ccc;
    border-radius: 4px;
    padding: 0 10px;
    ::v-deep .el-form-item{
      margin-bottom: 0;
      flex:1;
      .el-select{
        width: 90%;
      }
      &:last-child{
        .el-select{
          float: right;
        }
      }
    }
    .right-container-center{
      width: 32px;
      align-self: center;
      margin: 0 12px;
      text-align: center;
    }
  }
}
.inner-con-high{
  ::v-deep input{
    width: 340px;
  }
}
.step-icon{
  position: relative;
  padding-bottom: 13px;
  &:after{
    content:"";
    display: block;
    height: 1px;
    background:#eee;
    width: calc(100% - 40px);
    position: absolute;
    bottom: 0px;
    margin: 0 20px;
  }
  .step-circle{
    width: 30px;
    height: 30px;
    border-radius: 50%;
    background-color: $mainColor;
    text-align: center;
    line-height: 30px;
    color: #fff;
  }
  .step-description{
    font-size: 16px;
    margin-left: 20px;
    color: #333;
    &:before{
      content: '';
      display: inline-block;
      width: 5px;
      height: 26px;
      background-color: $mainColor;
      border-radius: 0px 5px 5px 0px;
      margin-right: 10px;
      transform: translateY(7px);
    }
  }
}
.organ-select{
  ::v-deep .el-input.is-disabled .el-input__inner{
    background-color: transparent;
  }
}

.header{
  //overflow: hidden;
  position: relative;
  height: 40px;
  margin-bottom: 20px;
  .organ-container{
    display: flex;
    justify-content: space-between;
    .organ-container-right{
      flex:1;
      border: 1px solid #ccc;
      border-radius: 4px;
      padding:0 10px;
      justify-content: space-between;
      .content-organ{
        align-self: center;
        font-size: 12px;
        background: #F6F6F6;
        color: $mainColor;
        padding: 5px 10px;
        border-radius: 2px;
        margin:0 12px;
        flex-shrink: 0;
      }
      ::v-deep .el-input{
        input{
          height: 38px;
        }
      }
    }
  }
  .line{
    border-top: 1px dotted #cccccc;
    width: 60%;
    margin:-20px auto 0;
    &-icon{
      width: 20px;
      height: 20px;
      border: 1px solid #666;
      position: absolute;
      left: 50%;
      transform: translate3d(-50%,0,0);
      top: 11px;
      background-color: #fff;
      font-size: 12px;
      line-height: 20px;
      text-align: center;
      color: #333;
    }
  }
  .organ{
    background-color: #fff;
    font-size: 14px;
    font-weight: bold;
    flex: 1;
    &:last-child{
      ::v-deep .el-select{
        float: right;
      }
    }
    ::v-deep .el-select{
      width: 90%;
    }
    span{
      width: 60px;
      display: inline-block;
      font-size: 16px;
    }
  }
  .popover-container{
    position: absolute;
    right: -22px;
    top: 12px;
  }
}
.item-row{
  margin: 0 auto;
  .item{
    margin-bottom: 22px;
  }
}
.center{
  width: 125px;
  // margin: 0 25px 0 60px;
  margin-left: 20px;
  text-align: center;
  font-size: 12px;
  color: #666;
}
.button-wrapper{
  padding-right: 20px;
  margin: 30px;
  text-align: right;
}
.option-item{
  display: flex;
  justify-content: space-between;
}
.option-label{
  max-width: 100%;
  // flex-shrink: 0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.option-desc{
  width: 80px;
  color: #999;
  margin-left: 10px;
}
.search-input{
  width: 300px;
}
</style>
