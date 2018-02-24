
<style lang="less">
    @import '../../styles/common.less';
</style>
<template><div>
<Card>
</Card>
<Card>
    <Row>
        <Col span="24" class="table-top-opt">
            <Button type="primary" icon="plus" size="small" @click="goFuncSave()">添加</Button>
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
                    <FormItem label="功能名称" prop="label">
                        <Input type="text" v-model="optForm.funcSave.name"></Input>
                    </FormItem>
                </Col>
                <Col span="11">
                    <FormItem label="功能Code" prop="name">
                        <Input type="text" v-model="optForm.funcSave.code"></Input>
                    </FormItem>
                </Col>
            </Row>
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
                <Col span="20">
                    <FormItem label="备注" prop="remark">
                        <Input type="textarea" v-model="optForm.funcSave.remark"></Input>
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
<Modal width="1000" v-model="optModal.genUpdate.show" @on-ok="optModal.genUpdate.show = false" title="生成代码">
    <Row>
        <Col style="margin-bottom: 15px;">
      框架： <Select v-model="optForm.genUpdate.framework" style="width:200px; margin-right: 20px;">
                <Option v-for="item in dict.framework" :value="item.value" :key="item.value">{{ item.label }}</Option>
            </Select>
      类型： <Select v-model="optForm.genUpdate.type" style="width:200px; margin-right: 20px;">
                <Option v-if="optForm.genUpdate.framework == 'iview-admin'" v-for="item in dict.typeIviewAdmin" :value="item.value" :key="item.value">{{ item.label }}</Option>
                <Option v-if="optForm.genUpdate.framework == 'lumen'" v-for="item in dict.typeLumen" :value="item.value" :key="item.value">{{ item.label }}</Option>
                <Option v-if="optForm.genUpdate.framework == 'database'" v-for="item in dict.typeRdb" :value="item.value" :key="item.value">{{ item.label }}</Option>
            </Select>
            <Button type="primary" :loading="optModal.genUpdate.gening" @click="doGenUpdate()">
                <span v-if="!optModal.genUpdate.gening">生成代码</span>
                <span v-else>生成中...</span>
            </Button>
            <Button type="primary" class="cbbtn" data-clipboard-target="#codeContainer" @click="$Message.info('已复制到剪切板')">复制代码</Button>
        </Col>
        <Col>
            <Input id="codeContainer" type="textarea" v-model="optModal.genUpdate.content" :rows="20"></Input>
            <Spin size="large" fix v-if="optModal.genUpdate.spinShow">生成中...</Spin>
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
                    title: '功能名称',
                    key: 'name',
                    width: 150,
                    sortable: true,
                    align: 'center'
                },
                {
                    title: '功能Code',
                    key: 'code',
                    width: 150,
                    sortable: true,
                    align: 'center'
                },
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
                    type: 'list',
                    framework: 'iview-admin',
                },
                funcSave: {
                    id: '',
                    relaAttr: '',
                    permissionTag: '',
                    remark: '',
                    name: '',
                    code: '',
                },
            },
            optModal: {
                genUpdate: {
                    show: false,
                    content: '',
                    spinShow: false,
                    gening: false,
                },
                funcSave: {
                    show: false,
                    title: '编辑',
                    loading: true
                },
            },
            dict: {
                framework: [
                    {
                        label: 'Lumen (PHP)',
                        value: 'lumen'
                    },
                    {
                        label: 'Iview-Admin (Vue.js)',
                        value: 'iview-admin'
                    },
                    {
                        label: 'RDB(Relation Database)',
                        value: 'database'
                    },
                ],
                typeIviewAdmin: [
                    {
                        label: 'list',
                        value: 'list'
                    },
                ],
                typeLumen: [
                    {
                        label: 'Controller',
                        value: 'controller'
                    },
                    {
                        label: 'Migration',
                        value: 'migration'
                    },
                ],
                typeRdb: [
                    {
                        label: 'Mysql',
                        value: 'mysql'
                    }
                ]
            },
        }
    },
    mounted () {
        const clipboard = new Clipboard('.cbbtn');
        this.optForm.pid = this.$route.params.id;
        this.page.current = 1;
        this.getTableData();
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
                name: 'attr_management',
                params: {
                    fid: id
                },
            });
        },
        goGenUpdate (id) {
            this.showModalGenUpdate(id);
        },
        showModalGenUpdate (id) {
            this.optForm.genUpdate.id = id;
            this.optModal.genUpdate.show = true;
            this.optModal.genUpdate.content = '';
        },
        doGenUpdate () {
            this.optModal.genUpdate.content = '';
            this.optModal.genUpdate.gening = true;
            this.optModal.genUpdate.spinShow = true;
            let _self = this;
            util.ajax.post('/gen/func/' + this.optForm.genUpdate.framework + '/' + this.optForm.genUpdate.type + '/' + this.optForm.genUpdate.id).then(res => {
                if (res.status === 200) {
                    if (res.data.code === '0') {
                        _self.optModal.genUpdate.content = res.data.content;
                        _self.optModal.genUpdate.gening = false;
                        _self.optModal.genUpdate.spinShow = false;
                    }
                }
            }).catch(err => {
                _self.optModal.genUpdate.gening = false;
                _self.optModal.genUpdate.spinShow = false;
                _self.$Message.error(res.data.message);
                console.log(err);
            });
        },
        goFuncSave (id) {
            this.showModalFuncSave(id);
        },
        showModalFuncSave (id) {
            if (id) {
                this.optModal.funcSave.title = '编辑';
                let _self = this;
                util.ajax.get('/func/detail?id=' + id).then(res => {
                    if (res.status === 200) {
                        if (res.data.code === "0") {
                            _self.optForm.funcSave.id = res.data.content.id;
                            _self.optForm.funcSave.relaAttr = res.data.content.relaAttr;
                            _self.optForm.funcSave.permissionTag = res.data.content.permissionTag;
                            _self.optForm.funcSave.remark = res.data.content.remark;
                            _self.optForm.funcSave.name = res.data.content.name;
                            _self.optForm.funcSave.code = res.data.content.code;
                        }
                    }
                    this.optModal.funcSave.loading = false;
                }).catch(err => {
                    this.optModal.funcSave.loading = false;
                    console.log(err);
                });
            } else {
                this.optModal.funcSave.loading = false;
                this.optModal.funcSave.title = '添加';
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
            this.optForm.funcSave.name = '';
            this.optForm.funcSave.code = '';
        },
        goOptDetail (id) {
            this.$router.push({
                name: 'opt_management',
                params: {
                    fid: id
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
    },
    watch: {
        //生命周期属性值是方法时，不可以使用箭头函数，无法获取this对象。因为箭头方法邦定父类上下文，生命周期属性属于this，邦定父类为undefined.
        "optForm.genUpdate.framework": function (curVal, oldVal) {
            if (curVal == "lumen") {
                this.optForm.genUpdate.type = "controller";
            } else if (curVal == "database") {
                this.optForm.genUpdate.type = "mysql";
            } else {
                this.optForm.genUpdate.type = "list";
            }
        },
    }
};
</script>
