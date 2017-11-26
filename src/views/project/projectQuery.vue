
<style lang="less">
    @import '../../styles/common.less';
</style>
<template><div>
<Card>
    <p slot="title">查询</p>
    <Form :model="optForm.projectQuery" :label-width="100" >
        <Row>
        </Row>
    </Form>
        <Row>
            <Col span="24" style="text-align: center;">
                <Button type="primary" icon="android-search" @click="doProjectQuery">查询</Button>
                <Button type="error" icon="android-refresh" @click="doProjectQueryClear">清空</Button>
                <Button type="primary" icon="plus" @click="goProjectAdd">添加</Button>
            </Col>
        </Row>
</Card>
<Modal width="800" v-model="optModal.projectAdd.show" :mask-closable="false" loading @on-ok="doProjectAdd" :title="optModal.projectAdd.title">
<Card :bordered="false" dis-hover>
    <Form :model="optForm.projectAdd" :label-width="100" >
        <Row>
        </Row>
    </Form>
</Card>
</Modal>
<Modal width="800" v-model="optModal.projectUpdate.show" :mask-closable="false" loading @on-ok="doProjectUpdate" :title="optModal.projectUpdate.title">
<Card :bordered="false" dis-hover>
    <Form :model="optForm.projectUpdate" :label-width="100" >
        <Row>
            <Col span="8" style="margin-bottom: -14px;">
                <FormItem label="项目标签" prop="label">
                    <Input type="text" v-model="optForm.projectUpdate.label"></Input>
                </FormItem>
            </Col>
            <Col span="8" style="margin-bottom: -14px;">
                <FormItem label="备注" prop="remark">
                    <Input type="textarea" v-model="optForm.projectUpdate.remark"></Input>
                </FormItem>
            </Col>
        </Row>
    </Form>
    <Spin size="large" fix v-if="optModal.projectUpdate.loading">
        <Icon type="load-c" size=18 class="spin-icon-loading"></Icon>
        <div>加载中</div>
    </Spin>
</Card>
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
import util from '@/libs/util';
export default {
    data () {
        return {
            optForm: {
                projectQuery: {
                },
                projectAdd: {
                },
                projectUpdate: {
                    label: '',
                    remark: ''
                },
            },
            optModal: {
                projectAdd: {
                    show: false,
                    title: '添加',
                },
                projectUpdate: {
                    show: false,
                    title: '修改',
                    loading: true
                },
            },
            columnsList:[
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
                                        this.goProjectDelete(params.row.id)
                                    }
                                }
                            }, '删除'),
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
                                        this.goProjectUpdate(params.row.id)
                                    }
                                }
                            }, '修改'),
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
                                        this.goFuncDetail(params.row.id)
                                    }
                                }
                            }, '功能'),
                        ])
                    }
                }            ],
            tableData: [],
            page: {
                total: 0,
                size: 10,
                current: 1
            },
            dict: {
            }
        }
    },
    mounted () {
        this.init()
    },
    methods: {
        init () {
            this.getProjectList()
        },
        changePage (num) {
            this.page.current = num
            this.getProjectList()
        },
        doProjectQuery () {
            this.page.current = 1
            this.getProjectList()
        },
        doProjectQueryClear () {
        },
        getProjectList () {
            let _self = this
            util.ajax.get('?page=' + this.page.current + '&limit=' + this.page.size, {params: this.getProjectQueryForm()}).then(res => {
                if (res.status === 200) {
                    if (res.data.result === 1) {
                        _self.tableData = res.data.content.data
                        _self.page.total = res.data.content.total
                        _self.page.size = res.data.content.per_page
                        _self.page.current = res.data.content.current_page
                    }
                }
            }).catch(err => {
                console.log(err)
            })
        },
        getProjectQueryForm () {
            return this.optForm.projectQuery
        },
        goProjectDelete (id) {
            let _self = this
            this.$Modal.confirm({
                title: '删除',
                content: '确定要删除？',
                loading: true,
                onOk: () => {
                    let _modal = this.$Modal
                    util.ajax.post('?id=' + id).then(res => {
                        if (res.status === 200) {
                            if (res.data.code === "0") {
                                _self.$Message.info(res.data.message)
                            } else {
                                _self.$Message.error(res.data.message)
                            }
                        }
                        _self.afterDoProjectDelete(res)
                        _modal.remove()
                    }).catch(function (err) {
                        console.log(err)
                        _self.afterDoProjectDelete(err)
                        _modal.remove()
                    })
                }
            })
        },
        afterDoProjectDelete (res) {
        },
        goProjectAdd () {
            this.showModalProjectAdd()
        },
        doProjectAdd () {
            let _self = this
            util.ajax.post('', this.optForm.projectAdd).then(function (res) {
                _self.optModal.projectAdd.show = false
                _self.$Message.info(res.data.message)
                _self.afterDoProjectAdd()
              }).catch(function (err) {
                _self.optModal.projectAdd.show = false
              })
        },
        afterDoProjectAdd (res) {
        },
        showModalProjectAdd () {
            this.optModal.projectAdd.show = true
        },
        goProjectUpdate (id) {
            this.showModalProjectUpdate(id)
        },
        doProjectUpdate () {
            let _self = this
            util.ajax.post('', this.optForm.projectUpdate).then(function (res) {
                _self.optModal.projectUpdate.show = false
                _self.$Message.info(res.data.message)
                _self.afterDoProjectUpdate()
              }).catch(function (err) {
                _self.optModal.projectUpdate.show = false
              })
        },
        afterDoProjectUpdate (res) {
        },
        showModalProjectUpdate (id) {
            let _self = this
            util.ajax.get('?id=' + id).then(res => {
                if (res.status === 200) {
                    if (res.data.result === 1) {
                        _self.optForm.projectUpdate.label = res.data.content.label
                        _self.optForm.projectUpdate.remark = res.data.content.remark
                    }
                }
                this.optModal.projectUpdate.loading = false
            }).catch(err => {
                console.log(err)
                this.optModal.projectUpdate.loading = false
            })
            this.optModal.projectUpdate.show = true
        },
        goFuncDetail (id) {
            this.$router.push({
                name: 'func_detail',
                params: {
                    id: id
                }
            })
        },
    }
}
</script>
