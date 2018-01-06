<style lang="less">
    @import '../../styles/common.less';
</style>
<template><div>
<Card>
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
            <Button type="primary" icon="plus" size="small"  @click="goDictSave()">添加</Button>
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
<Modal width="800" v-model="optModal.dictSave.show" :mask-closable="false" :loading="optModal.dictSave.okLoading" @on-ok="doDictSave" :title="optModal.dictSave.title">
    <Card :bordered="false" dis-hover>
        <Form ref="dictSaveForm" :model="optForm.dictSave" :label-width="100" :rules="optRule.dictSave">
            <Row>
                <Col span="11">
                    <FormItem label="字典名称" prop="name" required>
                        <Input type="text" v-model="optForm.dictSave.name"></Input>
                    </FormItem>
                </Col>
                <Col span="11">
                    <FormItem label="备注" prop="remark" >
                        <Input type="textarea" v-model="optForm.dictSave.remark"></Input>
                    </FormItem>
                </Col>
            </Row>
            <FormItem label="字典项" prop="dictItems">
                <Row v-for="(item, index) in optForm.dictSave.dictItems" :key="index">
                    <Col span="7" style="margin-right: 5px;">
                        <Input type="text" placeholder="请输入值" v-model="item.value"></Input>
                    </Col>
                    <Col span="7" style="margin-right: 5px;">
                        <Input type="text" placeholder="请输入名称" v-model="item.name"></Input>
                    </Col>
                    <Col span="7">
                        <Button v-if="optForm.dictSave.dictItems.length > 1" type="primary" size="small" icon="minus" @click="removeDictItemSave(index)"></Button>
                        <Button v-if="optForm.dictSave.dictItems.length - 1 == index" type="primary" size="small" icon="plus" @click="addDictItemSave()"></Button>
                    </Col>
                </Row>
            </FormItem>
        </Form>
        <Spin size="large" fix v-if="optModal.dictSave.loading">
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
                    title: '字典名称',
                    key: 'name',
                    width: 150,
                    sortable: true,
                    align: 'center'
                },
                {
                    title: '字典项',
                    key: 'dictItems',
                    align: 'center',
                    render: (h, params) => params.row.dictItems.map(dictItem => dictItem.value + "：" + dictItem.name + "，")
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
                dictSave: {
                    id: '',
                    name: '',
                    remark: '',
                    dictItems: [
                        {
                            name: '',
                            value: ''
                        }
                    ],
                },
            },
            optModal: {
                dictSave: {
                    show: false,
                    title: '字典',
                    okLoading: true,
                    loading: true
                },
            },
            optRule: {
                dictSave: {
                    name: [
                        { required: true, message: '字典名称不能为空', trigger: 'blur' },
                    ],
                },
            },
            dict: {
            },
        }
    },
    mounted () {
        this.optForm.pid = this.$route.params.id;
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
            };
            queryForm.page = this.page.current - 1;
            queryForm.size = this.page.size;
            queryForm.pid = this.optForm.pid;
            return queryForm;
        },
        getTableData () {
            let _self = this;
            util.ajax.get('/dict/list', { params: this.processQueryForm() }).then(res => {
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
            this.doQuery();
        },
        goDictSave (id) {
            this.showModalDictSave(id);
        },
        showModalDictSave (id) {
            if (id) {
                this.optModal.dictSave.title = '编辑字典';
                let _self = this;
                util.ajax.get('/dict/detail?id=' + id).then(res => {
                    if (res.status === 200) {
                        if (res.data.code === "0") {
                            _self.prepareDictSaveForm(res.data.content);
                        }
                    }
                    this.optModal.dictSave.loading = false;
                }).catch(err => {
                    this.optModal.dictSave.loading = false;
                    console.log(err);
                });
            } else {
                this.clearDictSaveForm ();
                this.optModal.dictSave.loading = false;
                this.optModal.dictSave.title = '添加字典';
            }
            this.optModal.dictSave.show = true;
        },
        doDictSave () {
            let _self = this;
            this.$refs.dictSaveForm.validate((valid) => {
                if (valid) {
                    util.ajax.post('/dict/save?pid=' + this.optForm.pid, this.processDictSaveForm()).then(res => {
                        _self.getTableData();
                        _self.clearDictSaveForm ();
                        _self.optModal.dictSave.show = false;
                        _self.$Message.info(res.data.message);
                    }).catch(err => {
                        console.log(err);
                        _self.optModal.dictSave.show = false;
                    });
                } else {
                    _self.optModal.dictSave.okLoading = false;
                    _self.$nextTick(() => _self.optModal.dictSave.okLoading = true);
                }
            });
        },
        prepareDictSaveForm (form) {
            this.optForm.dictSave.id = form.id;
            this.optForm.dictSave.name = form.name.toString();
            this.optForm.dictSave.remark = form.remark.toString();
            if (form.dictItems.length > 0) {
                this.optForm.dictSave.dictItems = form.dictItems;
            }
        },
        processDictSaveForm () {
            let form = {
                id: this.optForm.dictSave.id,
                name: this.optForm.dictSave.name,
                remark: this.optForm.dictSave.remark,
                dictItems: this.optForm.dictSave.dictItems,
            };
            return form;
        },
        clearDictSaveForm () {
            this.optForm.dictSave.id = '';
            this.optForm.dictSave.name = '';
            this.optForm.dictSave.remark = '';
            this.optForm.dictSave.dictItems = [
                {
                    name: '',
                    value:'',
                }
            ];
        },
        goDictDelete (id) {
            let _self = this;
            this.$Modal.confirm({
                title: '删除',
                content: '确定要删除？',
                loading: true,
                onOk: () => {
                    let _modal = this.$Modal;
                    util.ajax.post('/dict/delete?id=' + id).then(res => {
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
                                    this.doAndShowGenerate(params.row.id)
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
                                    this.goDictSave(params.row.id);
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
                                    this.goDictDelete(params.row.id);
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
        removeDictItemSave (index) {
            this.optForm.dictSave.dictItems.splice(index, 1);
        },
        addDictItemSave () {
            this.optForm.dictSave.dictItems.push({
                name: '',
                value: '',
            });
        },
    }
};
</script>