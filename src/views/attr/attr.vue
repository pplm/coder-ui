<template><div>
<Card>
    <Form :model="queryForm" :label-width="80">
        <Row>
            <Col span="6" style="margin-bottom: -15px;">
                <FormItem label="标签" prop="label">
                    <Input type="text" v-model="queryForm.label"></Input>
                </FormItem>
            </Col>
            <Col span="6" style="margin-bottom: -15px;">
                <FormItem label="名字" prop="name">
                    <Input type="text" v-model="queryForm.name"></Input>
                </FormItem>
            </Col>
            <Col span="6" style="margin-bottom: -15px;">
                <FormItem label="类型" prop="type">
                    <Input type="text" v-model="queryForm.type"></Input>
                </FormItem>
            </Col>
            <Col span="6" style="margin-bottom: -15px;">
                <FormItem label="长度" prop="length">
                    <Input type="text" v-model="queryForm.length"></Input>
                </FormItem>
            </Col>
            <Col span="6" style="margin-bottom: -15px;">
                <FormItem label="精度" prop="precise">
                    <Input type="text" v-model="queryForm.precise"></Input>
                </FormItem>
            </Col>
            <Col span="6" style="margin-bottom: -15px;">
                <FormItem label="默认值" prop="defaultValue">
                    <Input type="text" v-model="queryForm.defaultValue"></Input>
                </FormItem>
            </Col>
        </Row>
        <Row>
            <Col span="7" style="text-align: center; margin-bottom: -15px;">
                <FormItem>
                    <Button type="primary" icon="android-search" @click="doQuery">查询</Button>
                    <Button type="error" icon="android-refresh" @click="doClear">清空</Button>
                    <Button type="success" icon="archive">导出</Button>
                </FormItem>
            </Col>
        </Row>
    </Form>
</Card>
<Modal width="900" v-model="saveModal.show" loading @on-ok="doSave" :title="saveModal.title">
        <Form :model="saveForm" :label-width="80" >
        <Row>
            <Col span="8" style="margin-bottom: -15px;">
                <FormItem label="标签" prop="label">
                    <Input type="text" v-model="saveForm.label" placeholder="请输入标签"></Input>
                </FormItem>
            </Col>
            <Col span="8" style="margin-bottom: -15px;">
                <FormItem label="名字" prop="name">
                    <Input type="text" v-model="saveForm.name" placeholder="请输入名字"></Input>
                </FormItem>
            </Col>
            <Col span="8" style="margin-bottom: -15px;">
                <FormItem label="类型" prop="type">
                    <Select v-model="saveForm.type" clearable placeholder="请选择类型" style="width:174px">
                        <Option v-for="item in dict.type" :value="item.value" :key="item.value">{{ item.label }}</Option>
                    </Select>
                </FormItem>
            </Col>
            <Col span="8" style="margin-bottom: -15px;">
                <FormItem label="长度" prop="length">
                    <Input type="text" v-model="saveForm.length" placeholder="请输入长度"></Input>
                </FormItem>
            </Col>
            <Col span="8" style="margin-bottom: -15px;">
                <FormItem label="精度" prop="precise">
                    <Input type="text" v-model="saveForm.precise" placeholder="请输入精度"></Input>
                </FormItem>
            </Col>
            <Col span="8" style="margin-bottom: -15px;">
                <FormItem label="默认值" prop="defaultValue">
                    <Input type="text" v-model="saveForm.defaultValue" placeholder="请输入默认值"></Input>
                </FormItem>
            </Col>
            <Col span="8" style="margin-bottom: -15px;">
                <FormItem label="备注" prop="remark">
                    <Input type="textarea" v-model="saveForm.remark" placeholder="请输入备注"></Input>
                </FormItem>
            </Col>
            <Col span="24" style="margin-bottom: -15px;">
                <FormItem label="操作项" prop="optIds">
                    <CheckboxGroup v-model="saveForm.optIds">
                        <Checkbox v-for="opt in saveModal.opts" :key="opt.id" :label="opt.id">{{ opt.label }}</Checkbox>
                    </CheckboxGroup>
                </FormItem>
            </Col>
        </Row>
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
                title: '',
                opts: []
            },
            queryForm: {
                label: '',
                name: '',
                type: '',
                length: '',
                precise: '',
                defaultValue: '',
            },
            saveForm: {
                label: '',
                name: '',
                type: '',
                length: '',
                precise: '',
                defaultValue: '',
                textarea: '',
                id: '',
                optIds: [],
                opts: []
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
                    width: 200,
                    align: 'center',
                    render: (h, params) => {
                        return this.dict.type.filter(item => {
                            return params.row.type == item.value
                        }).map(item => {
                            return item.label
                        }) + '【' + params.row.type + '】'
                    },
                },
                {
                    title: '长度',
                    key: 'length',
                    align: 'center'
                },
                {
                    title: '精度',
                    key: 'precise',
                    align: 'center'
                },
                {
                    title: '默认值',
                    key: 'defaultValue',
                    align: 'center'
                },
                {
                    title: '备注',
                    key: 'remark',
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
                                         let argu = { order_id: params.row.order_id }
                                           //util.openNewPage(this, 'order_info', argu);
                                           this.$router.push({
                                               name: 'order_info',
                                               params: argu
                                           })
                                       }
                                   }
                            }, '详情'),
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
                        label: '文本（默认）',
                        value: 'text'
                    },
                    {
                        label: '日期时间',
                        value: 'datetime'
                    },
                    {
                        label: '枚举（字典）',
                        value: 'enum'
                    },
                    {
                        label: '日期',
                        value: 'date'
                    },
                    {
                        label: '文本域',
                        value: 'textarea'
                    },
                    {
                        label: '数值',
                        value: 'number'
                    },
                    {
                        label: '密码',
                        value: 'password'
                    }
                ]
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
            this.queryForm.name = ''
            this.queryForm.type = ''
            this.queryForm.length = ''
            this.queryForm.precise = ''
            this.queryForm.defaultValue = ''
            this.doQuery()
        },
        doSave () {
            let _self = this;
            this.saveForm.opts = this.saveForm.optIds.map(id => {
                return {
                    id: id
                }
            })
            util.ajax.post('/attr/save/' + this.fid, this.saveForm).then(res => {
                _self.saveModal.show = false
                _self.$Message.info('操作成功')
                if (_self.saveForm.id === '') {
                    _self.getList()
                } else {
                    _self.getList()
                }
              }).catch(err => {
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
                    util.ajax.post('/attr/delete/' + id).then(res => {
                        _modal.remove()
                        if (res.status === 200) {
                            if (res.data.code === "0") {
                                _self.$Message.info('操作成功')
                                _self.getList()
                            }
                        }
                    }).catch(err => {
                        _modal.remove()
                        console.log(err)
                    })
                }
            })
        },
        getList () {
            let _self = this
            util.ajax.get('/attr/list?fid=' + this.fid + '&page=' + this.page.num + '&size=' + this.page.size).then(res => {
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
            this.saveForm.optIds = []
            this.loadOpts()
            this.clearSaveForm()
            this.saveModal.title = '添加功能'
            this.saveModal.show = true
        },
        showModalUpdate(id) {
            let _self = this
            util.ajax.get('/attr/detail?id=' + id).then(res => {
                if (res.status === 200) {
                    if (res.data.code === "0") {
                        _self.saveForm = res.data.content
                        _self.loadOpts()
                        _self.saveForm.optIds = _self.saveForm.opts.map(opt => opt.id)
                    }
                }
            }).catch(err => {
                console.log(err)
              })
            this.saveModal.show = true
            this.saveModal.title = '修改功能'
        },
        loadOpts () {
            let _self = this
            util.ajax.get('/opt/listAll?fid=' + this.fid).then(res => {
                if (res.status === 200) {
                    if (res.data.code === "0") {
                        _self.saveModal.opts = res.data.content.filter(opt => opt.type == 'add' || opt.type == 'update' || opt.type == 'detail' || opt.type == 'query' || opt.type == 'list' || opt.type == 'save')
                    }
                }
            }).catch(err => {
                console.log(err)
            })
        },
        clearSaveForm () {
            this.saveForm.label = ''
            this.saveForm.name = ''
            this.saveForm.type = ''
            this.saveForm.length = ''
            this.saveForm.precise = ''
            this.saveForm.defaultValue = ''
            this.saveForm.id = ''
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