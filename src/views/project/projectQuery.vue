<style lang="less">
    @import '../../styles/common.less';
</style>
<template><div>
<Card>
    <p slot="title">查询</p>
    <Form :model="optForm.queryForm" :label-width="100">
        <Row>
            <Col span="11">
                <FormItem label="项目名称" prop="name">
                    <Input type="text" v-model="optForm.queryForm.name"></Input>
                </FormItem>
            </Col>
            <Col span="11">
                <FormItem label="项目Code" prop="code">
                    <Input type="text" v-model="optForm.queryForm.code"></Input>
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
            <Button type="primary" icon="plus" size="small"  @click="goProjectSave()">添加</Button>
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
                    <FormItem label="项目名称" prop="name" required>
                        <Input type="text" v-model="optForm.projectSave.name"></Input>
                    </FormItem>
                </Col>
                <Col span="11">
                    <FormItem label="项目Code" prop="code" required>
                        <Input type="text" v-model="optForm.projectSave.code"></Input>
                    </FormItem>
                </Col>
            </Row>
            <Row>
                <Col span="11">
                    <FormItem label="备注" prop="remark" >
                        <Input type="textarea" v-model="optForm.projectSave.remark"></Input>
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
<Modal width="1000" v-model="optModal.rdbgUpdate.show" @on-ok="optModal.rdbgUpdate.show = false" title="生成代码">
    <Row>
        <Col style="margin-bottom: 15px;">
    数据库： <Select v-model="optForm.rdbgUpdate.type" style="width:200px; margin-right: 20px;">
                <Option v-for="item in dict.type" :value="item.value" :key="item.value">{{ item.label }}</Option>
            </Select>
            <Button type="primary" :loading="optModal.rdbgUpdate.gening" @click="doRdbgUpdate()">
                <span v-if="!optModal.rdbgUpdate.gening">生成代码</span>
                <span v-else>生成中...</span>
            </Button>
            <Button type="primary" class="cbbtn" data-clipboard-target="#codeContainer" @click="$Message.info('已复制到剪切板')">复制代码</Button>
        </Col>
        <Col>
            <Input id="codeContainer" type="textarea" v-model="optModal.rdbgUpdate.content" :rows="20"></Input>
            <Spin size="large" fix v-if="optModal.rdbgUpdate.spinShow">生成中...</Spin>
        </Col>
    </Row>
</Modal>
</div></template>
<script>
import util from '@/libs/util';
import Clipboard from 'clipboard';

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
                    title: '项目名称',
                    key: 'name',
                    sortable: true,
                    align: 'center'
                },
                {
                    title: '项目Code',
                    key: 'code',
                    sortable: true,
                    align: 'center'
                },
                {
                    title: '备注',
                    key: 'remark',
                    sortable: true,
                    align: 'center'
                },
            ],
            optForm: {
                queryForm: {
                    name: '',
                    code: '',
                },
                rdbgUpdate: {
                    id: '',
                    type: 'mysql',
                },
                projectSave: {
                    id: '',
                    name: '',
                    code: '',
                    remark: '',
                },
            },
            optModal: {
                rdbgUpdate: {
                    show: false,
                    content: '',
                    spinShow: false,
                    gening: false,
                },
                projectSave: {
                    show: false,
                    title: '项目',
                    okLoading: true,
                    loading: true
                },
            },
            optRule: {
                projectSave: {
                    name: [
                        { required: true, message: '项目名称不能为空', trigger: 'blur' },
                    ],
                    code: [
                        { required: true, message: '项目Code不能为空', trigger: 'blur' },
                    ],
                },
            },
            dict: {
                type: [
                    {
                        label: 'Mysql',
                        value: 'mysql'
                    },
                ],
            },
        }
    },
    mounted () {
        this.page.current = 1;
        this.getTableData();
        this.init();
    },
    methods: {
        init () {
            this.setOpts();
        },
        processQueryForm () {
            let queryForm = {
                name: this.optForm.queryForm.name,
                code: this.optForm.queryForm.code,
            };
            queryForm.page = this.page.current - 1;
            queryForm.size = this.page.size;
            return queryForm;
        },
        getTableData () {
            let _self = this;
            util.ajax.get('/project/list', { params: this.processQueryForm() }).then(res => {
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
            this.optForm.queryForm.name = '';
            this.optForm.queryForm.code = '';
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
        goRdbgUpdate (id) {
            this.showModalRdbgUpdate(id);
        },
        showModalRdbgUpdate (id) {
            this.optForm.rdbgUpdate.id = id;
            this.optModal.rdbgUpdate.show = true;
            this.optModal.rdbgUpdate.content = '';
        },
        doRdbgUpdate () {
            this.optModal.rdbgUpdate.content = '';
            this.optModal.rdbgUpdate.gening = true;
            this.optModal.rdbgUpdate.spinShow = true;
            let _self = this;
            util.ajax.post('/gen/project/database/' + this.optForm.rdbgUpdate.type + '/' + this.optForm.rdbgUpdate.id).then(res => {
                if (res.status === 200) {
                    if (res.data.code === '0') {
                        _self.optModal.rdbgUpdate.content = res.data.content;
                        _self.optModal.rdbgUpdate.gening = false;
                        _self.optModal.rdbgUpdate.spinShow = false;
                    }
                }
            }).catch(err => {
                _self.optModal.rdbgUpdate.gening = false;
                _self.optModal.rdbgUpdate.spinShow = false;
                _self.$Message.error(res.data.message);
                console.log(err);
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
                    console.log(res);
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
                this.clearProjectSaveForm ();
                this.optModal.projectSave.loading = false;
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
            this.optForm.projectSave.name = form.name.toString();
            this.optForm.projectSave.code = form.code.toString();
            this.optForm.projectSave.remark = form.remark.toString();
        },
        processProjectSaveForm () {
            let form = {
                id: this.optForm.projectSave.id,
                name: this.optForm.projectSave.name,
                code: this.optForm.projectSave.code,
                remark: this.optForm.projectSave.remark,
            };
            return form;
        },
        clearProjectSaveForm () {
            this.optForm.projectSave.id = '';
            this.optForm.projectSave.name = '';
            this.optForm.projectSave.code = '';
            this.optForm.projectSave.remark = '';
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
                                    this.goRdbgUpdate(params.row.id);
                                }
                            }
                        };
                    },
                    label: 'RDBG'
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
                                    this.goDictDetail(params.row.id);
                                }
                            }
                        };
                    },
                    label: '字典'
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
                                    this.goProjectDelete(params.row.id);
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
