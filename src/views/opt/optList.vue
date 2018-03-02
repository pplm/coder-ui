<template><div>
<Card>
    <Form :model="queryForm" :label-width="100">
        <Row>
            <Col span="6">
                <FormItem label="操作" prop="label">
                    <Input type="text" v-model="queryForm.label"></Input>
                </FormItem>
            </Col>
            <Col span="6">
                <FormItem label="类型" prop="type">
                    <Select v-model="queryForm.type" clearable placeholder="请选择类型" style="width:174px">
                        <Option v-for="item in dict.type" :value="item.value" :key="item.value">{{ item.label }}</Option>
                    </Select>
                </FormItem>
            </Col>
            <Col span="6">
                <FormItem label="模式" prop="mode">
                    <Select v-model="queryForm.mode" clearable placeholder="请选择模式" style="width:174px">
                        <Option v-for="item in dict.mode" :value="item.value" :key="item.value">{{ item.label }}</Option>
                    </Select>
                </FormItem>
            </Col>
            <Col span="6">
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
                    <Button type="error" icon="android-refresh" @click="$router.go(-1)">返回</Button>
                </FormItem>
            </Col>
        </Row>
    </Form>
</Card>
<Modal width="900" v-model="saveModal.show" loading @on-ok="doSave" :title="saveModal.title">
    <Form :model="saveForm" :label-width="100" >
        <Row>
            <Col span="10">
                <FormItem label="操作名称" prop="name">
                    <Input type="text" v-model="saveForm.name"></Input>
                </FormItem>
            </Col>
            <Col span="10">
                <FormItem label="操作Code" prop="code">
                    <Input type="text" v-model="saveForm.code"></Input>
                </FormItem>
            </Col>
        </Row>
        <Row>
            <Col span="10">
                <FormItem label="类型" prop="type">
                    <Select v-model="saveForm.type" clearable placeholder="请选择类型" style="width:174px">
                        <Option v-for="item in dict.type" :value="item.value" :key="item.value">{{ item.label }}</Option>
                    </Select>
                </FormItem>
            </Col>
            <Col span="10">
                <FormItem label="模式" prop="mode">
                    <Select v-model="saveForm.mode" clearable placeholder="请选择模式" style="width:174px">
                        <Option v-for="item in dict.mode" :value="item.value" :key="item.value">{{ item.label }}</Option>
                    </Select>
                </FormItem>
            </Col>
        </Row>
        <Row>
            <Col span="10">
                <FormItem label="数据准备url" prop="preUrl">
                    <Input type="text" v-model="saveForm.preUrl"></Input>
                </FormItem>
            </Col>
            <Col span="10">
                <FormItem label="数据准备method" prop="preMethod">
                    <Select v-model="saveForm.preMethod" clearable placeholder="请选择method" style="width:174px">
                        <Option v-for="item in dict.preMethod" :value="item.value" :key="item.value">{{ item.label }}</Option>
                    </Select>
                </FormItem>
            </Col>
        </Row>
        <Row>
            <Col span="10">
                <FormItem label="执行url" prop="exeUrl">
                    <Input type="text" v-model="saveForm.exeUrl"></Input>
                </FormItem>
            </Col>
            <Col span="10">
                <FormItem label="执行method" prop="exeMethod">
                    <Select v-model="saveForm.exeMethod" clearable placeholder="请选择method" style="width:174px">
                        <Option v-for="item in dict.exeMethod" :value="item.value" :key="item.value">{{ item.label }}</Option>
                    </Select>
                </FormItem>
            </Col>
        </Row>
        <Row>
            <FormItem label="权限值" prop="permissionTag">
                <Col span="10">
                    <Input type="text" v-model="saveForm.permissionTag"></Input>
                </Col>
            </FormItem>
        </Row>
    </Form>
</Modal>
<Modal width="900" v-model="genModal.show" @on-ok="genModal.show = false" title="生成代码">
    <Row>
        <Col style="margin-bottom: 5px;">
            <Button type="primary" class="cbbtn" data-clipboard-target="#codeContainer" @click="$Message.info('已复制到剪切板')">复制代码</Button>
            <Button type="primary" @click="doAndShowGenerate()">重新生成</Button>

        </Col>
        <Col>
            <Input id="codeContainer" type="textarea" v-model="genModal.content" :rows="15"></Input>
            <Spin size="large" fix v-if="genModal.spinShow">生成中...</Spin>
        </Col>
    </Row>
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
import Clipboard from 'clipboard';
export default {
    data () {
        return {
            fid: '',
            saveModal: {
                show: false,
                title: ''
            },
            genModal: {
                show: false,
                content: '',
                spinShow: false,
                id: '',
                type: '',
            },
            queryForm: {
                name: '',
                code: '',
                type: '',
                mode: '',
                preUrl: '',
                exeUrl: ''
            },
            saveForm: {
                id: '',
                name: '',
                code: '',
                type: '',
                mode: '',
                preUrl: '',
                preMethod: '',
                exeUrl: '',
                exeMethod: '',
                permissionTag: ''
            },
            columnsList:[
                {
                    title: '操作名称',
                    key: 'name',
                    width: 110,
                    align: 'center'
                },
                {
                    title: '操作Code',
                    key: 'code',
                    width: 110,
                    align: 'center'
                },
                {
                    title: '类型',
                    key: 'type',
                    width: 110,
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
                    width: 80,
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
                    title: '数据准备method',
                    key: 'preMethod',
                    align: 'center',
                    render: (h, params) => {
                        return this.dict.preMethod.filter(item => {
                            return params.row.preMethod == item.value
                        }).map(item => {
                            return item.label
                        })
                    }
                },
                {
                    title: '执行url',
                    key: 'exeUrl',
                    align: 'center'
                },
                {
                    title: '执行method',
                    key: 'exeMethod',
                    align: 'center',
                    render: (h, params) => {
                        return this.dict.exeMethod.filter(item => {
                            return params.row.exeMethod == item.value
                        }).map(item => {
                            return item.label
                        })
                    }
                },
                {
                    title: '权限值',
                    key: 'permissionTag',
                    align: 'center'
                },
                {
                    title: '操作',
                    key: 'action',
                    align: 'center',
                    fixed: 'right',
                    width: 250,
                    render: (h, params) => {
                        let opts = [
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
                                            name: 'opt_attr_list',
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
                        ]
                        if (params.row.mode === "page") {
                            opts.splice(0, 0, h('Button', {
                                props: {
                                    type: 'primary',
                                    size: 'small'
                                },
                                style: {
                                    marginRight: '5px'
                                },
                                on: {
                                    click: () => {
                                        this.doAndShowGenerate({
                                            id: params.row.id,
                                            type: params.row.type
                                        })
                                    }
                                }
                            }, '生成'))
                        }
                        return h('div', opts)
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
                        label: '查询',
                        value: 'query'
                    },
                    {
                        label: '修改',
                        value: 'update'
                    },
                    {
                        label: '添加',
                        value: 'add'
                    },
                    {
                        label: '添加修改(合并)',
                        value: 'save'
                    },
                    {
                        label: '删除',
                        value: 'delete'
                    },
                    {
                        label: '导出',
                        value: 'export'
                    },
                    {
                        label: '结果',
                        value: 'list'
                    },
                    {
                        label: '详情',
                        value: 'detail'
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
                preMethod: [
                    {
                        label: 'GET',
                        value: 'get'
                    },
                    {
                        label: 'POST',
                        value: 'post'
                    },
                    {
                        label: 'PUT',
                        value: 'put'
                    },
                    {
                        label: 'DELETE',
                        value: 'delete'
                    },
                ],
                exeMethod: [
                    {
                        label: 'GET',
                        value: 'get'
                    },
                    {
                        label: 'POST',
                        value: 'post'
                    },
                    {
                        label: 'PUT',
                        value: 'put'
                    },
                    {
                        label: 'DELETE',
                        value: 'delete'
                    },
                ],
            }
        }
    },
    mounted () {
        const clipboard = new Clipboard('.cbbtn')
        this.init()
    },
    activated () {
        this.init()
    },
    methods: {
        doAndShowGenerate(obj) {
            this.genModal.show = true;
            this.genModal.content = '';
            this.genModal.spinShow = true;
            if (obj) {
                this.genModal.id = obj.id;
                this.genModal.type = obj.type;
            }
            let _self = this
            util.ajax.post('/gen/opt/iview-admin/' + this.genModal.type + '/' + this.genModal.id).then(res => {
                if (res.status === 200) {
                    if (res.data.code === "0") {
                        _self.genModal.content = res.data.content
                    } else {
                        _self.genModal.content = res.data.message
                    }
                    _self.genModal.spinShow = false
                }
            }).catch(err => {
                _self.genModal.content = err.message
                _self.genModal.spinShow = false
                console.log(err)
            })
        },
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
            util.ajax.post('/opt/save?fid=' + this.fid, this.saveForm).then(function (res) {
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
                    util.ajax.post('/opt/delete?id=' + id).then(function (res) {
                        _modal.remove()
                        if (res.status === 200) {
                            if (res.data.code === "0") {
                                _self.getList()
                            }
                            _self.$Message.info(res.data.message)
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
            this.saveForm.name = ''
            this.saveForm.code = ''
            this.saveForm.type = ''
            this.saveForm.mode = ''
            this.saveForm.preUrl = ''
            this.saveForm.preMethod =''
            this.saveForm.exeUrl = ''
            this.saveForm.exeMethod = ''
            this.saveForm.permissionTag = ''
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
