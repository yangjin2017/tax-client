<style lang="less">
  .operation {
    margin-bottom: 2vh;
  }
</style>
<template>
  <Row>
    <Col>
    <Card>
      <Row>
        <Form ref="form" :model="form" inline :label-width="90" class="search-form">
          <Form-item label="公司名称" prop="companyId">
            <Select v-model="form.companyId" filterable style="width: 200px" @on-change="changeCompany" label-in-value>
              <Option v-for="item in companyList" :value="item.id" :key="item.id" :label="item.name">
              </Option>
            </Select>
          </Form-item>
          <Form-item label="税务识别号码" prop="tin">
            <Input type="text" v-model="form.tin" readonly style="width: 200px" />
          </Form-item>
          <Form-item label="国家" prop="countryCode">
            <Select v-model="form.countryCode" disabled style="width:200px">
              <Option v-for="item in dictCountrys" :value="item.code" :key="item.id">{{ item.name }}</Option>
            </Select>
          </Form-item>
          <Form-item label="币种" prop="currency">
            <Select v-model="form.currency" disabled style="width:200px">
              <Option v-for="item in dictCurrencys" :value="item.code" :key="item.id">{{ item.name }}</Option>
            </Select>
          </Form-item>
          <!-- <Form-item label="申请人" prop="applicantName">
            <Input type="text" v-model="form.applicantName" clearable placeholder="请输入申请人姓名" style="width: 200px" />
          </Form-item> -->
          <Form-item label="备注" prop="remarks">
            <Input type="text" v-model="form.remarks" clearable placeholder="请输入备注" style="width: 200px" />
          </Form-item>
          <br/>
          <Form-item label="财务报表" prop="financialReport">
            <Upload action="/api/file/upload" :headers="{accessToken: accessToken}" name="file" :data="{materialTypeDict: 'FINANCE_REPORT'}" :show-upload-list="false" :on-success="financeUploadSuc">
              <Button icon="ios-cloud-upload-outline">上传文件</Button>
            </Upload>
          </Form-item>
        </Form>
        <Spin size="large" fix v-if="loading"></Spin>
      </Row>
      <Row class="operation">
        <!-- <Button @click="delAll" icon="md-trash">批量删除</Button> -->
        <Button @click="addColumn" icon="md-add">增加一栏</Button>
        <Button @click="submit('save')" >保存</Button>
        <Button @click="submit" >提交</Button>
        <circleLoading v-if="operationLoading"/>
      </Row>
      <!-- <Row>
        <Alert show-icon>
          已选择 <span class="select-count">{{selectCount}}</span> 项
          <a class="select-clear" @click="clearSelectAll">清空</a>
        </Alert>
      </Row> -->
      <Row>
        <Table ref="table" border :columns="columns" :data="data" @on-selection-change="changeSelect"></Table>
      </Row>
    </Card>
    </Col>
    <Modal
        v-model="showUploadModal"
        title="Common Modal dialog box title"
        @on-ok="uploadModalOk"
        @on-cancel="uploadModalCancel">
        <Form label-position="left" :label-width="100">
          <FormItem label="申报表">
            <Upload action="/api/file/upload" :headers="{accessToken: accessToken}" name="file" :data="{materialTypeDict: 'TAX_REPORT'}" :show-upload-list="false" :on-success="uploadSuc">
              <Button icon="ios-cloud-upload-outline">上传文件</Button>
              <div v-if="fileUploadForm.taxReturns">{{ fileUploadForm.taxReturns_name }}<Button @click.stop="filePriview(fileUploadForm.taxReturns)">预览</Button></div>
            </Upload>
          </FormItem>
          <FormItem label="完税申报表">
            <Upload action="/api/file/upload" :headers="{accessToken: accessToken}" name="file" :data="{materialTypeDict: 'DONE_TAX_REPORT'}" :show-upload-list="false" :on-success="uploadSuc">
              <Button icon="ios-cloud-upload-outline">上传文件</Button>
              <div v-if="fileUploadForm.paymentCertificate">{{ fileUploadForm.paymentCertificate_name }}</div>
            </Upload>
          </FormItem>
          <FormItem label="其它">
            <Upload action="/api/file/upload" :headers="{accessToken: accessToken}" name="file" :data="{materialTypeDict: 'OTHER'}" :show-upload-list="false" :on-success="uploadSuc">
              <Button icon="ios-cloud-upload-outline">上传文件</Button>
              <div v-if="fileUploadForm.otherUploadId">{{ fileUploadForm.otherUploadId_name }}</div>
            </Upload>
          </FormItem>
      </Form>
    </Modal>
  </Row>
</template>

<script>
import {
  getAllCompany,
  getDictListDataByType,
  getCompanyByName,
  taxAdd,
  previewFile
} from '@/api/index'
import { dictType } from '@/libs/constance.js'
import { getStore } from '@/libs/storage';
import circleLoading from "../../my-components/circle-loading.vue"
export default {
  name: 'taxApplication',
  data() {
    return {
      loading: false,
      operationLoading: false,
      accessToken: getStore('accessToken'),
      showUploadModal: false,
      form: {
        companyId: '',
        tin: '',
        countryCode: '',
        currency: '',
        applicantName: '',
        remarks: '',
        financialReport: ''
      },
      columns: [
        {
          title: '操作',
          width: 80,
          fixed: 'right',
          align: "center",
          render: (h, params) => {
            return h('Button', {
              props: {
                type: "error",
                size: "small"
              },
              on: {
                click: () => {
                  if (this.data.length > 1) {
                    this.data.splice(params.index, 1);
                  }
                }
              }
            }, '删除')
          }
        },
        {
          title: '所属期间',
          key: "taxPeriod",
          width: 160,
          align: 'center',
          // render: this.renderSelect
          render: this.renderDatePicker
        },
        {
          title: '税种',
          key: "taxDict",
          align: 'center',
          width: 160,
          render: this.renderSelect
        },
        {
          title: '应缴税额',
          key: "payableTax",
          align: 'center',
          width: 120,
          render: this.renderInput
        },
        {
          title: '应缴滞纳金',
          key: "lateFeePayable",
          align: 'center',
          width: 120,
          render: this.renderInput
        },
        {
          title: '申请缴纳税款',
          key: "applTaxPayment",
          align: 'center',
          width: 120,
          render: this.renderInput
          /* render: (h, params) => {
            let item = this.data[params.index];
            item[params.column.key] = parseFloat(item.payableTax) + parseFloat(item.lateFeePayable);
            return h('div', item[params.column.key])
          } */
        },
        {
          title: '缴款截止日期',
          key: "deadline",
          width: 160,
          align: 'center',
          render: this.renderDatePicker
        },
        {
          title: '实缴税额',
          key: "taxPaid",
          align: 'center',
          width: 120,
          render: this.renderInput
        },
        {
          title: '实缴滞纳金',
          key: "overduePayment",
          align: 'center',
          width: 120,
          render: this.renderInput
        },
        {
          title: '实际缴纳税款',
          align: 'center',
          width: 120,
          render: (h, params) => {
            let item = this.data[params.index];
            let num = parseFloat(item.taxPaid) + parseFloat(item.overduePayment);
            return h('div', num)
          }
        },
        {
          title: '实际缴纳日期',
          key: "paymentTime",
          align: 'center',
          width: 160,
          render: this.renderDatePicker
        },
        {
          title: '附件',
          key: "taxReturns",
          align: 'center',
          width: 180,
          render: (h, params) => {
            return h("div", [
                h(
                  "Button",
                  {
                    props: {
                      type: "primary",
                      size: "small"
                    },
                    style: {
                      marginRight: "5px"
                    },
                    on: {
                      click: () => {
                        let item = this.data[params.index];
                        let {taxReturns, paymentCertificate, otherUploadId} = item;
                        this.fileUploadForm = {
                          uploadColomunIndex: params.index,
                          taxReturns,
                          paymentCertificate,
                          otherUploadId
                        }
                        this.showUploadModal = true;
                      }
                    }
                  },
                  "上传"
                ),
                h(
                  "Button",
                  {
                    props: {
                      size: "small"
                    },
                    style: {
                      marginRight: "5px"
                    },
                    on: {
                      click: () => {
                        let item = this.data[params.index];
                        let {taxReturns, paymentCertificate, otherUploadId} = item;
                        this.fileUploadForm = {
                          uploadColomunIndex: params.index,
                          taxReturns,
                          paymentCertificate,
                          otherUploadId
                        }
                        this.showUploadModal = true;
                      }
                    }
                  },
                  "预览"
                ),
                h(
                  "Button",
                  {
                    props: {
                      type: "error",
                      size: "small"
                    },
                    on: {
                      click: () => {
                        // this.remove(params.row);
                      }
                    }
                  },
                  "删除"
                )
              ]);
          }
        },
        {
          title: '备注',
          key: "remarks",
          align: 'center',
          width: 160,
          render: this.renderInput
        },
      ],
      data: [],
      companyList: [],
      dictCountrys: [],
      dictCurrencys: [],
      dictTaxCategorys: [],
      dictTaxPayments: [],
      selectList: [],
      selectCount: 0,
      fileUploadForm: {
        uploadColomunIndex: null,
        taxReturns: '',
        paymentCertificate: '',
        otherUploadId: ''
      }
    }
  },
  methods: {
    init() {
      this.initPageData()
      this.initCompanyList()
      this.getDictData()
      this.addColumn()
    },
    initPageData() {
      let type = this.$route.params.type;
      if (!type) return;
      if (type === 'readyCommit') {
        let data = this.$route.params.params.details;
        data && data.map(item => {
          item.taxPeriod.replace(/-01$/, '');
        });
        this.data = data || [];
        let params = this.$route.params.params;
        this.form.companyId = params.companyId;
        this.form.tin = params.tin;
        this.form.countryCode = params.countryCode;
        this.form.currency = params.currency;
        this.form.remarks = params.remarks;
        this.form.financialReport = params.financialReport;
      }
    },
    /* 获取公司列表 */
    initCompanyList() {
      getAllCompany()
        .then(res => {
          this.companyList = res.data;
        })
    },
    /* 获取字典信息 */
    getDictData() {
      getDictListDataByType(dictType.country)
        .then(res => {
          this.dictCountrys = res.data;
        });
      getDictListDataByType(dictType.currency)
        .then(res => {
          this.dictCurrencys = res.data;
        });
      getDictListDataByType(dictType.taxCategory)
        .then(res => {
          this.dictTaxCategorys = res.data;
        })
      getDictListDataByType(dictType.taxPayment)
        .then(res => {
          this.dictTaxPayments = res.data;
        })
    },
    /* 删除 */
    delAll() {
      if (this.selectCount <= 0) {
        this.$Message.warning("您还未选择要删除的数据");
        return;
      }
      this.$Modal.confirm({
        title: "确认删除",
        content: "您确认要删除所选的 " + this.selectCount + " 条数据?",
        onOk: () => {
          let ids = "";
          this.selectList.forEach(function(e) {
            console.log(e);
            ids += e.id + ",";
          });
          /* ids = ids.substring(0, ids.length - 1);
          this.operationLoading = true;
          deleteLog(ids).then(res => {
            this.operationLoading = false;
            if (res.success === true) {
              this.$Message.success("删除成功");
              this.clearSelectAll();
              this.init();
            }
          }); */
        }
      });
    },
    /* 取消选中 */
    clearSelectAll() {
      this.$refs.table.$children[0].selectAll(false);
    },
    /* 选择 */
    changeSelect(e) {
      this.selectList = e;
      this.selectCount = e.length;
    },
    addColumn() {
      this.data.length > 0 || this.data.push({
        taxPeriod: '',
        taxDict: '',
        payableTax: 0,
        lateFeePayable: 0,
        applTaxPayment: '',
        deadline: '',
        taxPaid: 0,
        overduePayment: 0,
        paymentTime: '',
        taxReturns: '',
        remarks: ''
      });
    },
    /* 表格栏输入框渲染函数 */
    renderInput(h, params) {
      return h('Input', {
        props: {
          type: 'text',
          value: params.row[params.column.key]
        },
        on: {
          'on-change': e => {
            this.data[params.index][params.column.key] = e.target.value;
          }
        }
      })
    },
    /* 表格框下拉选择框渲染函数 */
    renderSelect(h, params) {
      const ArrayList = {
        taxPeriod: 'dictTaxPayments',   // 所属期间
        taxDict: 'dictTaxCategorys',   // 税种
      }
      return h('Select', {
        props: {
          value: params.row[params.column.key]
        },
        on: {
          'on-change': val => {
            this.data[params.index][params.column.key] = val;
          }
        }
      }, this[ArrayList[params.column.key]].map(item => {
          return h('Option', {
            props: {
              value: item.code,
              label: item.name
            }
          })
        })
      )
    },
    /* 表格日期选择框渲染函数 */
    renderDatePicker(h, params) {
      return h('DatePicker', {
        props: {
          type: params.column.key === 'taxPeriod' ? 'month' : 'date',
          value: params.row[params.column.key]
        },
        on: {
          'on-change': val => {
            this.data[params.index][params.column.key] = val;
          }
        }
      })
    },
    /* 选择公司后获取对应的税种等信息 */
    changeCompany(company) {
      this.loading = true;
      getCompanyByName({name: company.label})
        .then(res => {
          this.form.tin = res.data.tin;
          this.form.countryCode = res.data.countryCode;
          this.form.currency = res.data.currencyCode;
        })
        .finally(() => {
          this.loading = false;
        })
    },
    /* 财务报表上传成功 */
    financeUploadSuc(res) {
      this.form.financialReport = res.data.id;
    },
    /* 税金申请 - 文件上传 */
    uploadSuc(res) {
      let key = {
        'TAX_REPORT': 'taxReturns',
        'DONE_TAX_REPORT': 'paymentCertificate',
        'OTHER': 'otherUploadId'
      }[res.data.materialTypeDict];
      this.fileUploadForm[key] = res.data.id;
      this.fileUploadForm[key + '_name'] = res.data.oriName;
    },
    uploadModalOk() {
      let {uploadColomunIndex, taxReturns, paymentCertificate, otherUploadId} = this.fileUploadForm;
      this.data[uploadColomunIndex].taxReturns = taxReturns;
      this.data[uploadColomunIndex].paymentCertificate = paymentCertificate;
      this.data[uploadColomunIndex].otherUploadId = otherUploadId;
      this.showUploadModal = false;
    },
    uploadModalCancel() {
      this.fileUploadForm = {
        uploadColomunIndex: null,
        taxReturns: '',
        paymentCertificate: '',
        otherUploadId: ''
      }
      this.showUploadModal = false;
    },
    filePriview(name) {
      previewFile(name).then(res => {
        console.log(res);
      });
      return false;
    },
    /* 提交 */
    submit(save) {
      let params = {...this.form, details: this.data};
      // 所属期间字段显示月份，但提交后台需要精确值日
      params.details.map(item => {
        item.taxPeriod = item.taxPeriod + '-01';
      });
      params.executeType = save === 'save' ? 0 : 1;
      this.loading = true;
      taxAdd(params).then(res => {
        this.$Message.success('操作成功')
      }).finally(() => {
        this.loading = false;
      })
    }
  },
  mounted() {
    this.init();
  }
}
</script>
