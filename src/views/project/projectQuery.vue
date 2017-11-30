
<style lang="less">
    @import '../../styles/common.less';
</style>
<template><div>
<Card>
    <p slot="title">查询</p>
    <Form :model="optForm.queryForm" :label-width="100">
        <Row>
            <Col span="11">
                <FormItem label="项目标签" prop="label">
                    <Input type="text" v-model="optForm.queryForm.label"></Input>
                </FormItem>
            </Col>
            <Col span="11">
                <FormItem label="项目名称" prop="name">
                    <Input type="text" v-model="optForm.queryForm.name"></Input>
                </FormItem>
            </Col>
        </Row>
    </Form>
    <Row>
        <Col span="24" style="text-align: center;">
            <Button type="primary" icon="android-search" @click="doQuery">查询</Button>
            <Button type="error" icon="android-refresh" @click="doQueryClear">清空</Button>
        </Col>
    </Row>
</Card>
<Card>
    <Row>
        <Col span="24" class="table-top-opt">
            <Button type="primary" icon="plus" size="small" @click="goProjectSave">添加</Button>
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
<Modal width="800" v-model="optModal.projectSave.show" :mask-closable="false" :loading="optModal.projectSave.okLoading" @on-ok="doProjectSave" :title="optModal.projectSave.title">
    <Card :bordered="false" dis-hover>
        <Form ref="projectSaveForm" :model="optForm.projectSave" :label-width="100" :rules="optRule.projectSave">
            <Row>
                <Col span="11">
                    <FormItem label="备注" prop="remark" >
                        <Input type="textarea" v-model="optForm.projectSave.remark"></Input>
                    </FormItem>
                </Col>
                <Col span="11">
                    <FormItem label="项目标签" prop="label" required>
                        <Input type="text" v-model="optForm.projectSave.label"></Input>
                    </FormItem>
                </Col>
            </Row>
            <Row>
                <Col span="11">
                    <FormItem label="项目名称" prop="name" required>
                        <Input type="text" v-model="optForm.projectSave.name"></Input>
                    </FormItem>
                </Col>
            </Row>
        </Form>
        <Spin size="large" fix v-if="optModal.projectSave.loading">
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
                    title: '备注',
                    key: 'remark',
                    width: 150,
                    sortable: true,
                    align: 'center'
                },
                {
                    title: '项目标签',
                    key: 'label',
                    width: 150,
                    sortable: true,
                    align: 'center'
                },
                {
                    title: '项目名称',
                    key: 'name',
                    width: 150,
                    sortable: true,
                    align: 'center'
                },
            ],
            optForm: {
                queryForm: {
                    label: '',
                    name: '',
                },
                projectSave: {
                    id: '',
                    remark: '',
                    label: '',
                    name: '',
                },
            },
            optModal: {
                projectSave: {
                    show: false,
                    title: '项目',
                    okLoading: true,
                    loading: true
                },
            },
            optRule: {
                projectSave: {
                    label:[
                        { required: true, message: '项目标签不能为空', trigger: 'blur' },
                    ],
                    name:[
                        { required: true, message: '项目名称不能为空', trigger: 'blur' },
                    ],
                },
            },
            dict: {
            },
        }
    },
    activated () {
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
            return queryForm;
        },
        getTableData () {
            let _self = this;
            util.ajax.get('/project/list', { params: this.getQueryForm() }).then(res => {
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
        doQuery () {
            this.page.current = 1;
            this.getTableData();
        },
        doQueryClear () {
            this.optForm.queryForm.label = '';
            this.optForm.queryForm.name = '';
            this.doQuery();
        },
        goProjectDelete (id) {
            let _self = this;
            this.$Modal.confirm({
                title: '删除',
                content: '确定要删除？',
                loading: true,
                onOk: () => {
                    let _modal = this.$Modal;
                    util.ajax.post('/project/delete?id=' + id).then(res => {
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
        goFuncDetail (id) {
            this.$router.push({
                name: 'func_query',
                params: {
                    id: id
                },
            });
        },
        goProjectSave (id) {
            this.showModalProjectSave(id);
        },
        showModalProjectSave (id) {
            if (id) {
                this.optModal.projectSave.title = '编辑项目';
                let _self = this;
                util.ajax.get('/project/detail?id=' + id).then(res => {
                    if (res.status === 200) {
                        if (res.data.code === "0") {
                            _self.prepareProjectSaveForm(res.data.content);
                        }
                    }
                    this.optModal.projectSave.loading = false;
                }).catch(err => {
                    this.optModal.projectSave.loading = false;
                    console.log(err);
                });
            } else {
                this.optModal.projectSave.title = '添加项目';
            }
            this.optModal.projectSave.show = true;
        },
        doProjectSave () {
            let _self = this;
            this.$refs.projectSaveForm.validate((valid) => {
                if (valid) {
                    util.ajax.post('/project/save', this.processProjectSaveForm()).then(res => {
                        _self.getTableData();
                        _self.clearProjectSaveForm ();
                        _self.optModal.projectSave.show = false;
                        _self.$Message.info(res.data.message);
                    }).catch(err => {
                        console.log(err);
                        _self.optModal.projectSave.show = false;
                    });
                } else {
                    _self.optModal.projectSave.okLoading = false;
                    _self.$nextTick(() => _self.optModal.projectSave.okLoading = true);
                }
            });
        },
        prepareProjectSaveForm (form) {
            this.optForm.projectSave.id = form.id;
            this.optForm.projectSave.remark = form.remark;
            this.optForm.projectSave.label = form.label;
            this.optForm.projectSave.name = form.name;
        },
        processProjectSaveForm () {
            let form = {
                id: this.optForm.projectSave.id,
                remark: this.optForm.projectSave.remark,
                label: this.optForm.projectSave.label,
                name: this.optForm.projectSave.name,
            };
            return form;
        },
        clearProjectSaveForm () {
            this.optForm.projectSave.id = '';
            this.optForm.projectSave.remark = '';
            this.optForm.projectSave.label = '';
            this.optForm.projectSave.name = '';
        },
        goDictDetail (id) {
            this.$router.push({
                name: 'dict_query',
                params: {
                    id: id
                },
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
                                    this.goProjectDelete(params.row.id);
                                }
                            }
                        };
                    },
                    label: '删除'
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
                                    this.goFuncDetail(params.row.id);
                                }
                            }
                        };
                    },
                    label: '功能'
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
                                    this.goProjectSave(params.row.id);
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
                                    this.goDictDetail(params.row.id);
                                }
                            }
                        };
                    },
                    label: '字典'
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
