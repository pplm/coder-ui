<template><div>
<Card>
    <Form :model="queryForm" :label-width="100">
        <Row>
            <Col span="6" style="margin-bottom: -15px;">
                <FormItem label="操作" prop="label">
                    <Input type="text" v-model="queryForm.label"></Input>
                </FormItem>
            </Col>
            <Col span="6" style="margin-bottom: -15px;">
                <FormItem label="类型" prop="type">
                    <Select v-model="queryForm.type" clearable placeholder="请选择类型" style="width:174px">
                        <Option v-for="item in dict.type" :value="item.value" :key="item.value">{{ item.label }}</Option>
                    </Select>
                </FormItem>
            </Col>
            <Col span="6" style="margin-bottom: -15px;">
                <FormItem label="模式" prop="mode">
                    <Select v-model="queryForm.mode" clearable placeholder="请选择模式" style="width:174px">
                        <Option v-for="item in dict.mode" :value="item.value" :key="item.value">{{ item.label }}</Option>
                    </Select>
                </FormItem>
            </Col>
            <Col span="6" style="margin-bottom: -15px;">
                <FormItem label="执行url" prop="url">
                    <Input type="text" v-model="queryForm.url"></Input>
                </FormItem>
            </Col>
        </Row>
        <Row>
            <Col span="24" style="text-align: center; margin-bottom: -15px;">
                <FormItem>
                    <Button type="primary" icon="android-search" @click="doQuery">查询</Button>
                    <Button type="error" icon="android-refresh" @click="doClear">清空</Button>
                </FormItem>
            </Col>
        </Row>
    </Form>
</Card>
<Modal width="700" v-model="saveModal.show" loading @on-ok="doSave" :title="saveModal.title">
        <Form :model="saveForm" :label-width="80" >
        <Row>
        <Row>
            <Col span="10" style="margin-bottom: -15px;">
                <FormItem label="标签" prop="label">
                    <Input type="text" v-model="saveForm.label"></Input>
                </FormItem>
            </Col>
            <Col span="10" style="margin-bottom: -15px;">
                <FormItem label="名字" prop="name">
                    <Input type="text" v-model="saveForm.name"></Input>
                </FormItem>
            </Col>
            <Col span="10" style="margin-bottom: -15px;">
                <FormItem label="类型" prop="type">
                    <Select v-model="saveForm.type" clearable placeholder="请选择类型" style="width:174px">
                        <Option v-for="item in dict.type" :value="item.value" :key="item.value">{{ item.label }}</Option>
                    </Select>
                </FormItem>
            </Col>
            <Col span="10" style="margin-bottom: -15px;">
                <FormItem label="模式" prop="mode">
                    <Select v-model="saveForm.mode" clearable placeholder="请选择模式" style="width:174px">
                        <Option v-for="item in dict.mode" :value="item.value" :key="item.value">{{ item.label }}</Option>
                    </Select>
                </FormItem>
            </Col>
            <Col span="10" style="margin-bottom: -15px;">
                <FormItem label="数据准备url" prop="preUrl">
                    <Input type="text" v-model="saveForm.preUrl"></Input>
                </FormItem>
            </Col>
            <Col span="10" style="margin-bottom: -15px;">
                <FormItem label="执行url" prop="exeUrl">
                    <Input type="text" v-model="saveForm.exeUrl"></Input>
                </FormItem>
            </Col>
        </Row>        </Row>
        </Form>
</Modal>
<Card>
<Button type="primary" icon="plus" style="margin-bottom: 5px; margin-top: -10px; text-align:right;" @click="showModalAdd">添加</Button>
    <Table :columns="columnsList" :data="tableData" border></Table>
    <div style="text-align: right; padding-top: 10px;">
        <Page :total="page.total" :current="page.current" :page-size="page.size" show-total show-elevator @on-change="changePage">
            总计 {{page.total}} 条
        </Page>
    </div>
</Card>
</div></template>
<script>
import util from '@/libs/util';
export default {
    data () {
        return {
            fid: '',
            saveModal: {
                show: false,
                title: ''
            },
            queryForm: {
                label: '',
                name: '',
                type: '',
                mode: '',
                preUrl: '',
                exeUrl: ''
            },
            saveForm: {
                id: '',
                label: '',
                name: '',
                type: '',
                mode: '',
                preUrl: '',
                exeUrl: ''
            },
            columnsList:[
                {
                    title: '标签',
                    key: 'label',
                    align: 'center'
                },
                {
                    title: '名字',
                    key: 'name',
                    align: 'center'
                },
                {
                    title: '类型',
                    key: 'type',
                    align: 'center',
                    render: (h, params) => {
                        return this.dict.type.filter(item => {
                            return params.row.type == item.value
                        }).map(item => {
                            return item.label
                        })
                    }
                },
                {
                    title: '模式',
                    key: 'mode',
                    align: 'center',
                    render: (h, params) => {
                        return this.dict.mode.filter(item => {
                            return params.row.mode == item.value
                        }).map(item => {
                            return item.label
                        })
                    }
                },
                {
                    title: '数据准备url',
                    key: 'preUrl',
                    align: 'center'
                },
                {
                    title: '执行url',
                    key: 'exeUrl',
                    align: 'center'
                },
                {
                    title: '操作',
                    key: 'action',
                    fixed: 'right',
                    width: 200,
                    render: (h, params) => {
                        return h('div', [
                            h('Button', {
                                props: {
                                    type: 'primary',
                                      icon: 'document',
                                      size: 'small'
                                },
                                style: {
                                      marginRight: '5px'
                                },
                                on: {
                                    click: () => {
                                        this.$router.push({
                                            name: 'opt_attr_management',
                                            params: {
                                                oid: params.row.id,
                                                fid: this.fid
                                            }
                                        })
                                    }
                                }
                            }, '属性'),
                            h('Button', {
                                props: {
                                    type: 'primary',
                                      icon: 'compose',
                                      size: 'small'
                                },
                                style: {
                                      marginRight: '5px'
                                },
                                on: {
                                      click: () => {
                                          this.showModalUpdate(params.row.id)
                                       }
                                   }
                            }, '编辑'),
                            h('Button', {
                                props: {
                                    type: 'primary',
                                      icon: 'ios-trash-outline',
                                      size: 'small'
                                },
                                style: {
                                      marginRight: '5px'
                                },
                                on: {
                                      click: () => {
                                          this.doDelete(params.row.id)
                                       }
                                   }
                            }, '删除')
                        ])
                    }
                }
            ],
            tableData: [],
            page: {
                total: 0,
                size: 10,
                current: 1,
                num: 0
            },
            dict: {
                type: [
                    {
                        label: '查询条件',
                        value: 'query'
                    },
                    {
                        label: '编辑（修改）操作',
                        value: 'update'
                    },
                    {
                        label: '添加操作',
                        value: 'add'
                    },
                    {
                        label: '删除操作',
                        value: 'delete'
                    },
                    {
                        label: '导出操作',
                        value: 'export'
                    },
                    {
                        label: '结果列表',
                        value: 'list'
                    }
                ],
                mode: [
                    {
                        label: '弹窗',
                        value: 'modal'
                    },
                    {
                        label: '页面',
                        value: 'page'
                    },
                    {
                        label: '提示',
                        value: 'tip'
                    },
                    {
                        label: '开关',
                        value: 'switch'
                    },
                    {
                        label: '列表',
                        value: 'list'
                    },
                    {
                        label: 'Excel',
                        value: 'excel'
                    }
                ],
            }        
        }
    },
    activated () {
        this.init()
    },
    methods: {
        init () {
            this.fid = this.$route.params.fid
            this.clearPage()
            this.getList()
        },
        doQuery() {
            this.clearPage()
            this.getList()
        },
        doClear () {
            this.queryForm.label = ''
            this.queryForm.type = ''
            this.queryForm.mode = ''
            this.queryForm.url = ''
            this.doQuery()
        },
        doSave () {
            let _self = this;
            util.ajax.post('/opt/save/' + this.fid, this.saveForm).then(function (res) {
                _self.saveModal.show = false
                _self.$Message.info('操作成功')
                _self.doQuery()
              }).catch(function (err) {
                _self.saveModal.show = false
              })
        },
        doDelete (id) {
            let _self = this
            this.$Modal.confirm({
                title: '删除',
                content: '确定要删除？',
                loading: true,
                onOk: () => {
                    let _modal = this.$Modal
                    util.ajax.post('/opt/delete/' + id).then(function (res) {
                        _modal.remove()
                        if (res.status === 200) {
                            if (res.data.code === "0") {
                                _self.$Message.info('操作成功')
                                _self.getList()
                            }
                        }
                    }).catch(function (err) {
                        _modal.remove()
                        console.log(err)
                    })
                }
            })
        },
        getList () {
            let _self = this
            util.ajax.get('/opt/list?fid=' + this.fid + '&page=' + this.page.num + '&size=' + this.page.size).then(res => {
                if (res.status === 200) {
                    if (res.data.code === "0") {
                        _self.tableData = res.data.content.content
                        _self.page.total = res.data.content.totalElements
                        _self.page.size = res.data.content.size
                        _self.page.current = res.data.content.number + 1
                    }
                }
            }).catch(err => {
                console.log(err)
            })
        },
        showModalAdd() {
            this.clearSaveForm()
            this.saveModal.title = '添加功能'
            this.saveModal.show = true
        },
        showModalUpdate(id) {
            let _self = this
            util.ajax.get('/opt/detail?id=' + id).then(res => {
                if (res.status === 200) {
                    if (res.data.code === "0") {
                        _self.saveForm = res.data.content
                        console.log(_self.saveForm)
                    }
                }
            }).catch(err => {
                console.log(err)
              })
            this.saveModal.show = true
            this.saveModal.title = '修改功能'
        },
        clearSaveForm () {
            this.saveForm.id = ''
            this.saveForm.label = ''
            this.saveForm.name = ''
            this.saveForm.type = ''
            this.saveForm.mode = ''
            this.saveForm.exeUrl = ''
            this.saveForm.preUrl = ''
        },
        clearPage () {
            this.page.size = 10
            this.page.current = 1
            this.page.num = 0
        },
        changePage (number) {
            this.page.num = number - 1
            this.getList()
        }
    }
}
</script>