
<style lang="less">
    @import '../../styles/common.less';
</style>
<template><div>
<Card>
    <p slot="title">字典信息列表</p>
    <Form :model="optForm.dictQuery" :label-width="100" >
        <Row>
            <Col span="8" style="margin-bottom: -14px;">
                <FormItem label="标签" prop="label">
                    <Input type="text" v-model="optForm.dictQuery.label"></Input>
                </FormItem>
            </Col>
        </Row>
    </Form>
        <Row>
            <Col span="24" style="text-align: center;">
                <Button type="primary" icon="android-search" @click="doDictQuery">查询</Button>
                <Button type="error" icon="android-refresh" @click="doDictQueryClear">清空</Button>
                <Button type="primary" icon="plus" @click="goDictAdd">添加</Button>
            </Col>
        </Row>
</Card>
<Modal width="800" v-model="optModal.dictAdd.show" :mask-closable="false" loading @on-ok="doDictAdd" :title="optModal.dictAdd.title">
<Card :bordered="false" dis-hover>
    <Form :model="optForm.dictAdd" :label-width="100" >
        <Row>
            <Col span="10" style="margin-bottom: -14px;">
                <FormItem label="标签" prop="label">
                    <Input type="text" v-model="optForm.dictAdd.label"></Input>
                </FormItem>
            </Col>
        </Row>
        <Row>
            <Col span="24" style="margin-bottom: -14px;">
                <FormItem label="备注" prop="remark">
                    <Input type="textarea" style="width: 600px;" v-model="optForm.dictAdd.remark"></Input>
                </FormItem>
            </Col>
        </Row>
        <FormItem label="字典项" prop="dictItems">
            <Row v-for="(item, index) in optForm.dictAdd.dictItems" :key="index">
                <Col span="7" style="margin-right: 5px;">
                    <Input type="text" placeholder="请输入键" v-model="item.label"></Input>
                </Col>
                <Col span="7" style="margin-right: 5px;">
                    <Input type="text" placeholder="请输入值" v-model="item.value"></Input>
                </Col>
                <Col span="7">
                    <Button v-if="optForm.dictAdd.dictItems.length > 1" type="primary" size="small" icon="minus" @click="removeDictItemAdd(index)"></Button>
                    <Button v-if="optForm.dictAdd.dictItems.length - 1 == index" type="primary" size="small" icon="plus" @click="addDictItemAdd()"></Button>
                </Col>
            </Row>
        </FormItem>
    </Form>
</Card>
</Modal>
<Modal width="800" v-model="optModal.dictUpdate.show" :mask-closable="false" loading @on-ok="doDictUpdate" :title="optModal.dictUpdate.title">
<Card :bordered="false" dis-hover>
    <Form :model="optForm.dictUpdate" :label-width="100" >
        <Row>
            <Col span="10" style="margin-bottom: -14px;">
                <FormItem label="标签" prop="label">
                    <Input type="text" v-model="optForm.dictUpdate.label"></Input>
                </FormItem>
            </Col>
        </Row>
        <Row>
            <Col span="24" style="margin-bottom: -14px;">
                <FormItem label="备注" prop="remark">
                    <Input type="textarea" style="width: 600px;" v-model="optForm.dictUpdate.remark"></Input>
                </FormItem>
            </Col>
        </Row>
        <FormItem label="字典项" prop="dictItems">
            <Row v-for="(item, index) in optForm.dictUpdate.dictItems" :key="index">
                <Col span="7" style="margin-right: 5px;">
                    <Input type="text" placeholder="请输入键" v-model="item.label"></Input>
                </Col>
                <Col span="7" style="margin-right: 5px;">
                    <Input type="text" placeholder="请输入值" v-model="item.value"></Input>
                </Col>
                <Col span="7">
                    <Button v-if="optForm.dictUpdate.dictItems.length > 1" type="primary" size="small" icon="minus" @click="removeDictItemUpdate(index)"></Button>
                    <Button v-if="optForm.dictUpdate.dictItems.length - 1 == index" type="primary" size="small" icon="plus" @click="addDictItemUpdate"></Button>
                </Col>
            </Row>
        </FormItem>
    </Form>
    <Spin size="large" fix v-if="optModal.dictUpdate.loading">
        <Icon type="load-c" size=18 class="spin-icon-loading"></Icon>
        <div>加载中</div>
    </Spin>
</Card>
</Modal>
<Modal width="900" v-model="optModal.genModal.show" @on-ok="optModal.genModal.show = false" title="生成代码">
    <Row>
        <Col style="margin-bottom: 5px;">
            <Button type="primary" class="cbbtn" data-clipboard-target="#codeContainer" @click="$Message.info('已复制到剪切板')">复制代码</Button>
        </Col>
        <Col>
            <Input id="codeContainer" type="textarea" v-model="optModal.genModal.content" :rows="15"></Input>
            <Spin size="large" fix v-if="optModal.genModal.spinShow">生成中...</Spin>
        </Col>
    </Row>
</Modal>
<Card>
    <Table :columns="columnsList" :data="tableData" border></Table>
    <div style="text-align: right; padding-top: 10px;">
        <Page :total="page.total" :current="page.current" :page-size="page.size" show-total show-elevator @on-change="changePage">
            总计 {{page.total}} 条
        </Page>
    </div>
</Card>
</div></template>
<script>
import util from '@/libs/util'
import Clipboard from 'clipboard'
export default {
    data () {
        return {
            optForm: {
                dictQuery: {
                    label: ''
                },
                dictAdd: {
                    label: '',
                    remark: '',
                    dictItems: [
                        {
                            label: '',
                            value: ''
                        }
                    ]
                },
                dictUpdate: {
                    label: '',
                    remark: '',
                    dictItems: [
                        {
                            label: '',
                            value: ''
                        }
                    ]
                },
            },
            optModal: {
                dictAdd: {
                    show: false,
                    title: '添加字典',
                },
                dictUpdate: {
                    show: false,
                    title: '编辑字典',
                    loading: true
                },
                genModal: {
                    show: false,
                    content: '',
                    spinShow: false,
                },
            },
            columnsList:[
                {
                    title: '标签',
                    key: 'label',
                    sortable: true,
                    align: 'center'
                },
                {
                    title: '字典项',
                    key: 'dictItems',
                    align: 'center',
                    render: (h, params) => params.row.dictItems.map(dictItem => dictItem.value + ": " + dictItem.label + " ")
                },
                {
                    title: '备注',
                    key: 'remark',
                    sortable: true,
                    align: 'center'
                },
                {
                    title: '操作',
                    key: 'action',
                    fixed: 'right',
                    align: 'center',
                    width: 240,
                    render: (h, params) => {
                        return h('div', [
                            h('Button', {
                                props: {
                                    type: 'primary',
                                    size: 'small'
                                },
                                style: {
                                    marginRight: '5px'
                                },
                                on: {
                                    click: () => {
                                        this.doAndShowGenerate(params.row.id)
                                    }
                                }
                            }, '生成'),
                            h('Button', {
                                props: {
                                    type: 'primary',
                                    size: 'small'
                                },
                                style: {
                                    marginRight: '5px'
                                },
                                on: {
                                    click: () => {
                                        this.goDictUpdate(params.row.id)
                                    }
                                }
                            }, '编辑'),
                            h('Button', {
                                props: {
                                    type: 'primary',
                                    size: 'small'
                                },
                                style: {
                                    marginRight: '5px'
                                },
                                on: {
                                    click: () => {
                                        this.goDictDelete(params.row.id)
                                    }
                                }
                            }, '删除'),
                        ])
                    }
                }            ],
            tableData: [],
            page: {
                total: 0,
                size: 10,
                current: 1,
                num: 0
            },
            dict: {
            }
        }
    },
    mounted () {
        this.init()
    },
    methods: {
        doAndShowGenerate(id) {
            this.optModal.genModal.show = true
            this.optModal.genModal.content = ''
            this.optModal.genModal.spinShow = true
            let _self = this
            util.ajax.post('/gen/vue/dict/' + id).then(res => {
                if (res.status === 200) {
                    if(res.data.code === "0") {
                        _self.optModal.genModal.content = res.data.content
                    } else {
                        _self.optModal.genModal.content = res.data.message
                    }
                    _self.optModal.genModal.spinShow = false
                }
            }).catch(err => {
                _self.optModal.genModal.content = err.message
                _self.optModal.genModal.spinShow = false
                console.log(err)
            })
        },
        init () {
            this.getDictList()
        },
        changePage (num) {
            this.page.num = num - 1
            this.getDictList()
        },
        doDictQuery () {
            this.page.current = 1
            this.getDictList()
        },
        doDictQueryClear () {
            this.optForm.dictQuery.label = ''
        },
        getDictList () {
            let _self = this
            util.ajax.get('/dict/list?page=' + this.page.num + '&size=' + this.page.size, {params: this.getDictQueryForm()}).then(res => {
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
        getDictQueryForm () {
            return this.optForm.dictQuery
        },
        goDictAdd () {
            this.showModalDictAdd()
        },
        doDictAdd () {
            let _self = this
            util.ajax.post('/dict/save', this.optForm.dictAdd).then(function (res) {
                _self.optModal.dictAdd.show = false
                _self.$Message.info(res.data.message)
                _self.afterDoDictAdd()
              }).catch(function (err) {
                _self.optModal.dictAdd.show = false
              })
        },
        afterDoDictAdd (res) {
            this.getDictList()
        },
        showModalDictAdd () {
            this.optModal.dictAdd.show = true
        },
        goDictUpdate (id) {
            this.showModalDictUpdate(id)
        },
        doDictUpdate () {
            let _self = this
            console.log(this.optForm.dictUpdate)
            util.ajax.post('/dict/save', this.optForm.dictUpdate).then(function (res) {
                _self.optModal.dictUpdate.show = false
                _self.$Message.info(res.data.message)
                _self.afterDoDictUpdate()
              }).catch(function (err) {
                _self.optModal.dictUpdate.show = false
              })
        },
        afterDoDictUpdate (res) {
            this.getDictList()
        },
        showModalDictUpdate (id) {
            let _self = this
            util.ajax.get('/dict/detail?id=' + id).then(res => {
                if (res.status === 200) {
                    if (res.data.code === "0") {
                        _self.optForm.dictUpdate.id = res.data.content.id
                        _self.optForm.dictUpdate.label = res.data.content.label
                        _self.optForm.dictUpdate.remark = res.data.content.remark
                        if (res.data.content.dictItems.length > 0) {
                            _self.optForm.dictUpdate.dictItems = res.data.content.dictItems
                        }
                    }
                }
                this.optModal.dictUpdate.loading = false
            }).catch(err => {
                console.log(err)
                this.optModal.dictUpdate.loading = false
            })
            this.optModal.dictUpdate.show = true
        },
        removeDictItemAdd (index) {
            this.optForm.dictAdd.dictItems.splice(index, 1)
        },
        addDictItemAdd () {
            this.optForm.dictAdd.dictItems.push({
                label: '',
                value: ''
            })
        },
        removeDictItemUpdate (index) {
            this.optForm.dictUpdate.dictItems.splice(index, 1)
        },
        addDictItemUpdate () {
            this.optForm.dictUpdate.dictItems.push({
                label: '',
                value: ''
            })
        },
        goDictDelete (id) {
            let _self = this
            this.$Modal.confirm({
                title: '删除',
                content: '确定要删除？',
                loading: true,
                onOk: () => {
                    let _modal = this.$Modal
                    util.ajax.post('/dict/delete?id=' + id).then(res => {
                        if (res.status === 200) {
                            if (res.data.code === "0") {
                                _self.$Message.info(res.data.message)
                            } else {
                                _self.$Message.error(res.data.message)
                            }
                        }
                        _self.afterDoDictDelete(res)
                        _modal.remove()
                    }).catch(function (err) {
                        console.log(err)
                        _self.afterDoDictDelete(err)
                        _modal.remove()
                    })
                }
            })
        },
        afterDoDictDelete (res) {
            this.getDictList()
        },
    }
}
</script>
