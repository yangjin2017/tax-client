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
          <Form-item label="公司名称" prop="company">
            <Select v-model="form.company" filterable style="width: 200px" @on-change="changeCompany" label-in-value>
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
          <Form-item label="申请人" prop="applicantName">
            <Input type="text" v-model="form.applicantName" clearable placeholder="请输入申请人姓名" style="width: 200px" />
          </Form-item>
          <Form-item label="备注" prop="remarks">
            <Input type="text" v-model="form.remarks" clearable placeholder="请输入备注" style="width: 200px" />
          </Form-item>
          <br/>
          <Form-item label="财务报表" prop="financialReport">
            <Upload action="//jsonplaceholder.typicode.com/posts/">
              <Button icon="ios-cloud-upload-outline">上传文件</Button>
            </Upload>
          </Form-item>
        </Form>
        <Spin size="large" fix v-if="loading"></Spin>
      </Row>
      <Row class="operation">
        <!-- <Button @click="delAll" icon="md-trash">批量删除</Button> -->
        <Button @click="addColumn" icon="md-add">增加一栏</Button>
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
  </Row>
</template>

<script>
import {
  getAllCompany,
  getDictListDataByType,
  getCompanyByName,
  taxAdd
} from '@/api/index'
import { dictType } from '@/libs/constance.js'
import circleLoading from "../../my-components/circle-loading.vue"
export default {
  name: 'taxApplication',
  data() {
    return {
      loading: false,
      operationLoading: false,
      form: {
        company: '',
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
          render: (h, params) => {
            let item = this.data[params.index];
            item[params.column.key] = parseFloat(item.payableTax) + parseFloat(item.lateFeePayable);
            return h('div', item[params.column.key])
          }
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
                        // this.edit(params.row);
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
                        // this.disable(params.row);
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
      selectCount: 0
    }
  },
  methods: {
    init() {
      this.initCompanyList()
      this.getDictData()
      this.addColumn()
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
      this.data.push({
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
          'on-blur': e => {
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
    submit() {
      let params = {...this.form, details: this.data};
      params.details.map(item => {
        item.taxPeriod = item.taxPeriod + '-01';
      })
      this.loading = true;
      taxAdd(params).then(res => {
        
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
