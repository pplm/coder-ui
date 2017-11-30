
<style lang="less">
    @import '../../styles/common.less';
</style>
<template><div>
<Card>
</Card>
<Card>
    <Row>
        <Col span="24" class="table-top-opt">
            <Button type="primary" icon="plus" size="small" @click="goFuncSave">添加</Button>
        </Col>
    </Row>
    <Row>
        <Col span="24">
            <Table :columns="columns" :data="tableData" border></Table>
        </Col>
    </Row>
    <Row>
        <Col span="24" class="table-bottom-pageable">
            <Page :total="page.total" :current="page.current" :page-size="page.size" show-total show-elevator size="small" @on-change="changePage">总计 {{page.total}} 条</Page>
        </Col>
    </Row>
    
</Card>
<Modal width="800" v-model="optModal.genUpdate.show" :mask-closable="false" loading @on-ok="doGenUpdate" :title="optModal.genUpdate.title">
    <Card :bordered="false" dis-hover>
        <Form :model="optForm.genUpdate" :label-width="100">
            <Row>
            </Row>
        </Form>
        <Spin size="large" fix v-if="optModal.genUpdate.loading">
            <Icon type="load-c" size=18 class="spin-icon-loading"></Icon>
            <div>加载中</div>
        </Spin>
    </Card>
</Modal>
<Modal width="800" v-model="optModal.funcSave.show" :mask-closable="false" loading @on-ok="doFuncSave" :title="optModal.funcSave.title">
    <Card :bordered="false" dis-hover>
        <Form :model="optForm.funcSave" :label-width="100">
            <Row>
                <Col span="11">
                    <FormItem label="关联属性" prop="relaAttr">
                        <Input type="text" v-model="optForm.funcSave.relaAttr"></Input>
                    </FormItem>
                </Col>
                <Col span="11">
                    <FormItem label="权限值" prop="permissionTag">
                        <Input type="text" v-model="optForm.funcSave.permissionTag"></Input>
                    </FormItem>
                </Col>
            </Row>
            <Row>
                <Col span="11">
                    <FormItem label="备注" prop="remark">
                        <Input type="textarea" v-model="optForm.funcSave.remark"></Input>
                    </FormItem>
                </Col>
                <Col span="11">
                    <FormItem label="功能标签" prop="label">
                        <Input type="text" v-model="optForm.funcSave.label"></Input>
                    </FormItem>
                </Col>
            </Row>
            <Row>
                <Col span="11">
                    <FormItem label="功能名称" prop="name">
                        <Input type="text" v-model="optForm.funcSave.name"></Input>
                    </FormItem>
                </Col>
            </Row>
        </Form>
        <Spin size="large" fix v-if="optModal.funcSave.loading">
            <Icon type="load-c" size=18 class="spin-icon-loading"></Icon>
            <div>加载中</div>
        </Spin>
    </Card>
</Modal>
</div></template>
<script>
import util from '@/libs/util';
export default {
    data () {
        return {
            tableData: [],
            page: {
                total: 0,
                size: 10,
                current: 1,
                num: 0
            },
            columns: [
                {
                    title: '关联属性',
                    key: 'relaAttr',
                    width: 150,
                    sortable: true,
                    align: 'center'
                },
                {
                    title: '权限值',
                    key: 'permissionTag',
                    width: 150,
                    sortable: true,
                    align: 'center'
                },
                {
                    title: '备注',
                    key: 'remark',
                    width: 150,
                    sortable: true,
                    align: 'center'
                },
                {
                    title: '功能标签',
                    key: 'label',
                    width: 150,
                    sortable: true,
                    align: 'center'
                },
                {
                    title: '功能名称',
                    key: 'name',
                    width: 150,
                    sortable: true,
                    align: 'center'
                },
            ],
            optForm: {
                pid: '',
                queryForm: {
                },
                genUpdate: {
                    id: '',
                },
                funcSave: {
                    id: '',
                    relaAttr: '',
                    permissionTag: '',
                    remark: '',
                    label: '',
                    name: '',
                },
            },
            optModal: {
                genUpdate: {
                    show: false,
                    title: '生成',
                    loading: true
                },
                funcSave: {
                    show: false,
                    title: '编辑',
                    loading: true
                },
            },
            dict: {
            },
        }
    },
    activated () {
        this.optForm.pid = this.$route.params.id
        this.page.current = 1;
        this.getTableData();
    },
    mounted () {
        this.init();
    },
    methods: {
        init () {
            this.setOpts();
        },
        getQueryForm () {
            let queryForm = this.optForm.queryForm;
            queryForm.page = this.page.current - 1;
            queryForm.size = this.page.size;
            queryForm.pid = this.optForm.pid;
            return queryForm;
        },
        getTableData () {
            let _self = this;
            util.ajax.get('/func/list', { params: this.getQueryForm() }).then(res => {
                if (res.status === 200) {
                    if (res.data.code === "0") {
                        if (res.data.content.content.length == 0 && _self.page.current > 1 ) {
                            _self.page.current = _self.page.current - 1;
                            _self.getTableData();
                        } else {
                            _self.tableData = res.data.content.content;
                            _self.page.total = res.data.content.totalElements;
                            _self.page.size = res.data.content.size;
                            _self.page.current = res.data.content.number + 1;
                        }
                    } else {
                        _self.$Message.error(res.data.message);
                    }
                }
            }).catch(err => {
                console.log(err);
            });
        },
        changePage (number) {
            this.page.current = number;
            this.getTableData();
        },
        goAttrDetail (id) {
            this.$router.push({
                name: 'attr_query',
                params: {
                    id: id
                },
            });
        },
        goGenUpdate (id) {
            this.showModalGenUpdate(id);
        },
        showModalGenUpdate (id) {
            let _self = this;
            util.ajax.get('?id=' + id).then(res => {
                if (res.status === 200) {
                    if (res.data.code === "0") {
                        _self.optForm.genUpdate.id = res.data.content.id;
                    }
                }
                this.optModal.genUpdate.loading = false;
            }).catch(err => {
                this.optModal.genUpdate.loading = false;
                console.log(err);
            })
            this.optModal.genUpdate.show = true
        },
        doGenUpdate () {
            let _self = this;
            util.ajax.post('', this.optForm.genUpdate).then(res => {
                _self.getTableData();
                _self.optModal.genUpdate.show = false;
                _self.$Message.info(res.data.message);
            }).catch(err => {
                console.log(err);
                _self.optModal.genUpdate.show = false;
            });
        },
        goFuncSave (id) {
            this.showModalFuncSave(id);
        },
        showModalFuncSave (id) {
            if (id) {
                this.optModal.funcSave.title = '编辑编辑';
                let _self = this;
                util.ajax.get('/func/detail?id=' + id).then(res => {
                    if (res.status === 200) {
                        if (res.data.code === "0") {
                            _self.optForm.funcSave.id = res.data.content.id;
                            _self.optForm.funcSave.relaAttr = res.data.content.relaAttr;
                            _self.optForm.funcSave.permissionTag = res.data.content.permissionTag;
                            _self.optForm.funcSave.remark = res.data.content.remark;
                            _self.optForm.funcSave.label = res.data.content.label;
                            _self.optForm.funcSave.name = res.data.content.name;
                        }
                    }
                    this.optModal.funcSave.loading = false;
                }).catch(err => {
                    this.optModal.funcSave.loading = false;
                    console.log(err);
                });
            } else {
                this.optModal.funcSave.title = '添加编辑';
            }
            this.optModal.funcSave.show = true;
        },
        doFuncSave () {
            let _self = this;
            util.ajax.post('/func/save?pid=' + this.optForm.pid, this.optForm.funcSave).then(res => {
                _self.getTableData();
                _self.clearFuncSaveForm ();
                _self.optModal.funcSave.show = false;
                _self.$Message.info(res.data.message);
            }).catch(err => {
                console.log(err);
                _self.optModal.funcSave.show = false;
            });
        },
        clearFuncSaveForm () {
            this.optForm.funcSave.id = '';
            this.optForm.funcSave.relaAttr = '';
            this.optForm.funcSave.permissionTag = '';
            this.optForm.funcSave.remark = '';
            this.optForm.funcSave.label = '';
            this.optForm.funcSave.name = '';
        },
        goOptDetail (id) {
            this.$router.push({
                name: 'opt_query',
                params: {
                    id: id
                },
            });
        },
        goDeleteDelete (id) {
            let _self = this;
            this.$Modal.confirm({
                title: '删除',
                content: '确定要删除？',
                loading: true,
                onOk: () => {
                    let _modal = this.$Modal;
                    util.ajax.post('/func/delete?id=' + id).then(res => {
                        if (res.status === 200) {
                            if (res.data.code === "0") {
                                _self.getTableData();
                                _self.$Message.info(res.data.message);
                            } else {
                                _self.$Message.error(res.data.message);
                            }
                        }
                        _modal.remove();
                    }).catch(err => {
                        _modal.remove();
                        console.log(err);
                    });
                }
            });
        },
        setOpts() {
            let opts = [
                {
                    widget: 'Button',
                    attrs: params => {
                        return {
                            props: {
                                type: 'primary',
                                size: 'small'
                            },
                            style: {
                                marginRight: '5px',
                                display: 'inline',
                            },
                            on: {
                                click: () => {
                                    this.goAttrDetail(params.row.id);
                                }
                            }
                        };
                    },
                    label: '属性'
                },
                {
                    widget: 'Button',
                    attrs: params => {
                        return {
                            props: {
                                type: 'primary',
                                size: 'small'
                            },
                            style: {
                                marginRight: '5px',
                                display: 'inline',
                            },
                            on: {
                                click: () => {
                                    this.goGenUpdate(params.row.id);
                                }
                            }
                        };
                    },
                    label: '生成'
                },
                {
                    widget: 'Button',
                    attrs: params => {
                        return {
                            props: {
                                type: 'primary',
                                size: 'small'
                            },
                            style: {
                                marginRight: '5px',
                                display: 'inline',
                            },
                            on: {
                                click: () => {
                                    this.goFuncSave(params.row.id);
                                }
                            }
                        };
                    },
                    label: '修改'
                },
                {
                    widget: 'Button',
                    attrs: params => {
                        return {
                            props: {
                                type: 'primary',
                                size: 'small'
                            },
                            style: {
                                marginRight: '5px',
                                display: 'inline',
                            },
                            on: {
                                click: () => {
                                    this.goOptDetail(params.row.id);
                                }
                            }
                        };
                    },
                    label: '操作项'
                },
                {
                    widget: 'Button',
                    attrs: params => {
                        return {
                            props: {
                                type: 'primary',
                                size: 'small'
                            },
                            style: {
                                marginRight: '5px',
                                display: 'inline',
                            },
                            on: {
                                click: () => {
                                    this.goDeleteDelete(params.row.id);
                                }
                            }
                        };
                    },
                    label: '删除'
                },
            ];
            let optCount = opts.filter(opt => opt.attrs().style.display !== 'none').length;
            if (optCount > 0) {
                this.columns.push({
                    title: '操作',
                    key: 'action',
                    fixed: 'right',
                    align: 'center',
                    width: 60 * optCount,
                    render: (h, params) => {
                        return h('div', opts.map(optItem => {
                            return h(optItem.widget, optItem.attrs(params), optItem.label);
                        }));
                    }
                });
            }
        },
    }
};
</script>
