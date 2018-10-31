<template>
  <Row>
    <Col>
    <Card>
      <Row>
        <Form ref="searchForm" :model="searchForm" inline :label-width="70" class="search-form">
          <Form-item label="公司名称" prop="userName">
            <Input type="text" v-model="searchForm.userName" clearable placeholder="请输入公司名称" style="width: 200px" />
          </Form-item>
          <Form-item label="税务识别号码" prop="department">
            <!-- <Cascader v-model="selectDep" :data="department" :load-data="loadData" @on-change="handleChangeDep" change-on-select filterable placeholder="请选择或输入搜索部门" style="width: 200px"></Cascader> -->
          </Form-item>
          <Form-item label="国家" prop="tel">
            <Input type="text" v-model="searchForm.tel" clearable placeholder="请输入手机号" style="width: 200px" />
          </Form-item>
          <Form-item label="币种" prop="email">
            <Input type="text" v-model="searchForm.email" clearable placeholder="请输入邮箱" style="width: 200px" />
          </Form-item>
          <Form-item label="申请人" prop="sex">
            <Select v-model="searchForm.sex" placeholder="请选择" clearable style="width: 200px">
              <Option value="0">男</Option>
              <Option value="1">女</Option>
            </Select>
          </Form-item>
          <Form-item label="备注" prop="type">
            <Select v-model="searchForm.type" placeholder="请选择" clearable style="width: 200px">
              <Option value="0">普通用户</Option>
              <Option value="1">管理员</Option>
            </Select>
          </Form-item>
          <!-- <Form-item style="margin-left:-35px;" class="br">
                        <Button @click="handleSearch" type="primary" icon="ios-search">搜索</Button>
                        <Button @click="handleReset">重置</Button>
                        <a class="drop-down" @click="dropDown">{{dropDownContent}}
                            <Icon :type="dropDownIcon"></Icon>
                        </a>
                    </Form-item> -->
        </Form>
      </Row>
      <Row>
        <Tables border :columns="columns" v-model="data" editable></Tables>
        <!-- <Table :loading="loading" border :columns="columns" :data="data" sortable="custom" @on-sort-change="changeSort" @on-selection-change="showSelect" ref="table"></Table>
        <Table :columns="exportColumns" :data="exportData" ref="exportTable" style="display:none"></Table> -->
      </Row>
    </Card>
    </Col>
  </Row>
</template>

<script>
import Tables from '../../my-components/tables/tables'
export default {
  name: 'tax-table',
  components: {
    Tables
  },
  data() {
    return {
      searchForm: {
        userName: '',
        departmentId: '',
        tel: '',
        email: '',
        sex: '',
        type: '',
        status: '',
        pageNumber: 1,
        pageSize: 10,
        sort: 'createTime',
        order: 'desc',
        startDate: '',
        endDate: ''
      },
      columns: [
        {
          title: '所属期间',
          key: "realName",
          align: 'center'
        },
        {
          title: '税种',
          key: "realName",
          editable: true,
          align: 'center'
        },
        {
          title: '应缴税额',
          key: "realName",
          align: 'center'
        },
        {
          title: '应缴滞纳金',
          key: "realName",
          align: 'center'
        },
        {
          title: '申请缴纳税款',
          key: "realName",
          align: 'center'
        },
        {
          title: '缴款截止日期',
          key: "realName",
          align: 'center'
        },
        {
          title: '实缴税额',
          key: "realName",
          align: 'center'
        },
        {
          title: '实缴滞纳金',
          key: "realName",
          align: 'center'
        },
        {
          title: '实际缴纳税款',
          key: "realName",
          align: 'center'
        },
        {
          title: '实际缴纳日期',
          key: "realName",
          align: 'center'
        },
        {
          title: '附件',
          key: "realName",
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
          key: "realName",
          align: 'center'
        },
      ],
      data: [{
        realName: '11'
      },{
        realName: '11'
      }]
    }
  }
}
</script>
