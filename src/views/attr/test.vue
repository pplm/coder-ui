<template><div>
<Card>
    <Form :model="queryForm" :label-width="80">
        <Row>
            <Col span="6" style="margin-bottom: -15px;">
                <FormItem label="手机号" prop="mobile">
                    <Input type="text" v-model="queryForm.mobile"></Input>
                </FormItem>
            </Col>
            <Col span="6" style="margin-bottom: -15px;">
                <FormItem label="昵称" prop="nick_name">
                    <Input type="text" v-model="queryForm.nick_name"></Input>
                </FormItem>
            </Col>
            <Col span="6" style="margin-bottom: -15px;">
                <FormItem label="城市" prop="CityName">
                    <Input type="text" v-model="queryForm.CityName"></Input>
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
<Modal width="700" v-model="saveModal.show" loading @on-ok="doSave" :title="saveModal.title">
        <Form :model="saveForm" :label-width="80" >
        <Row>
            <Col span="10" style="margin-bottom: -15px;">
                <FormItem label="手机号" prop="mobile">
                    <Input type="text" v-model="saveForm.mobile" placeholder="请输入手机号"></Input>
                </FormItem>
            </Col>
            <Col span="10" style="margin-bottom: -15px;">
                <FormItem label="昵称" prop="nick_name">
                    <Input type="text" v-model="saveForm.nick_name" placeholder="请输入昵称"></Input>
                </FormItem>
            </Col>
            <Col span="10" style="margin-bottom: -15px;">
                <FormItem label="城市" prop="CityName">
                    <Input type="text" v-model="saveForm.CityName" placeholder="请输入城市"></Input>
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
            saveModal: {
                show: false,
                title: ''
            },
            queryForm: {
                mobile: '',
                nick_name: '',
                CityName: ''
            },
            saveForm: {
                id: '',
                mobile: '',
                nick_name: '',
                CityName: ''
            },
            columnsList:[
                {
                    title: '手机号',
                    key: 'mobile',
                    align: 'center'
                },
                {
                    title: '昵称',
                    key: 'nick_name',
                    align: 'center'
                },
                {
                    title: '城市',
                    key: 'CityName',
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
            }
        }
    },
    mounted () {
        this.init()
    },
    methods: {
        init () {
            this.clearPage()
            this.getList()
        },
        doQuery() {
            this.clearPage()
            this.getList()
        },
        doClear () {
            this.queryForm.mobile = ''
            this.queryForm.nick_name = ''
            this.queryForm.CityName = ''
            this.doQuery()
        },
        doSave () {
            let _self = this;
            util.ajax.post('/MemberManagement/MemberList/save', this.saveForm).then(function (res) {
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
                    util.ajax.post('/MemberManagement/MemberList/delete/' + id).then(function (res) {
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
            util.ajax.get('/MemberManagement/MemberList/list?page=' + this.page.num + '&size=' + this.page.size).then(function (res) {
                if (res.status === 200) {
                    if (res.data.code === "0") {
                        _self.tableData = res.data.data.content
                        _self.page.total = res.data.data.totalElements
                        _self.page.size = res.data.data.size
                        _self.page.current = res.data.data.number + 1
                    }
                }
            }).catch(function (error) {
                console.log(error)
              })
        },
        showModalAdd() {
            this.clearSaveForm()
            this.saveModal.title = '添加功能'
            this.saveModal.show = true
        },
        showModalUpdate(id) {
            let _self = this
            util.ajax.get('/MemberManagement/MemberList/detail?id=' + id).then(function (res) {
                if (res.status === 200) {
                    if (res.data.code === "0") {
                        _self.saveForm = res.data.data
                    }
                }
            }).catch(function (error) {
                console.log(error)
              })
            this.saveModal.show = true
            this.saveModal.title = '修改功能'
        },
        clearSaveForm () {
            this.saveForm.id = ''
            this.saveForm.mobile = ''
            this.saveForm.nick_name = ''
            this.saveForm.CityName = ''
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