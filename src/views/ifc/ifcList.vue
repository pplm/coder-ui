<style lang="less">
    @import '../../styles/common.less';
</style>
<template><div>
<Card>
    <Row>
        <Col span="24" class="table-top-opt">
            <Button type="primary" icon="plus" size="small" @click="goIfcSave()">添加</Button>
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
<Modal width="800" v-model="optModal.ifcSave.show" :mask-closable="false" :loading="optModal.ifcSave.okLoading" @on-ok="doIfcSave()" @on-cancel="cancelIfcSave()" :title="optModal.ifcSave.title">
    <Card :bordered="false" dis-hover>
        <Form ref="ifcSaveForm" :model="optForm.ifcSave" :label-width="100" :rules="optRule.ifcSave">
            <Row>
                <Col span="11">
                    <FormItem label="接口编号" prop="sn" >
                        <Input type="text" v-model="optForm.ifcSave.sn"></Input>
                    </FormItem>
                </Col>
                <Col span="11">
                    <FormItem label="接口名称" prop="name" required>
                        <Input type="text" v-model="optForm.ifcSave.name"></Input>
                    </FormItem>
                </Col>
            </Row>
            <Row>
                <Col span="11">
                    <FormItem label="接口方法" prop="method" >
                        <Select v-model="optForm.ifcSave.method" clearable placeholder="请选择接口方法">
                            <Option v-for="item in dict.method" :value="item.value" :key="item.value">{{ item.label }}</Option>
                        </Select>
                    </FormItem>
                </Col>
                <Col span="11">
                    <FormItem label="数据协议类型" prop="dataProtocol" >
                        <Select v-model="optForm.ifcSave.dataProtocol" placeholder="请选择数据协议类型">
                            <Option v-for="item in dict.dataProtocol" :value="item.value" :key="item.value">{{ item.label }}</Option>
                        </Select>
                    </FormItem>
                </Col>
            </Row>
            <Row>
                <Col span="11">
                    <FormItem label="接口url" prop="url" required>
                        <Input type="text" v-model="optForm.ifcSave.url"></Input>
                    </FormItem>
                </Col>
            </Row>
        </Form>
        <Spin size="large" fix v-if="optModal.ifcSave.loading">
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
                    title: '接口编号',
                    key: 'sn',
                    width: 150,
                    sortable: true,
                    align: 'center'
                },
                {
                    title: '接口名称',
                    key: 'name',
                    width: 150,
                    sortable: true,
                    align: 'center'
                },
                {
                    title: '接口方法',
                    key: 'method',
                    width: 150,
                    sortable: true,
                    render: (h, params) => {
                        return this.dict.method.filter(item => params.row.method == item.value).map(item => item.label);
                    },
                    align: 'center'
                },
                {
                    title: '数据协议类型',
                    key: 'dataProtocol',
                    width: 150,
                    sortable: true,
                    render: (h, params) => {
                        return this.dict.dataProtocol.filter(item => params.row.dataProtocol == item.value).map(item => item.label);
                    },
                    align: 'center'
                },
                {
                    title: '接口url',
                    key: 'url',
                    sortable: true,
                    align: 'center'
                },
            ],
            optForm: {
                fid: '',
                queryForm: {
                },
                ifcSave: {
                    id: '',
                    sn: '',
                    name: '',
                    method: '',
                    dataProtocol: 'json',
                    url: '',
                },
            },
            optModal: {
                ifcSave: {
                    show: false,
                    title: '接口',
                    okLoading: true,
                    loading: true
                },
            },
            optRule: {
                ifcSave: {
                    name: [
                        { required: true, message: '接口名称不能为空', trigger: 'blur' },
                    ],
                    url: [
                        { required: true, message: '接口url不能为空', trigger: 'blur' },
                    ],
                },
            },
            dict: {
                method: [
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
                dataProtocol: [
                    {
                        label: 'JSON',
                        value: 'json'
                    },
                    {
                        label: 'XML',
                        value: 'xml'
                    },
                ],
            },
        }
    },
    mounted () {
        this.optForm.fid = this.$route.params.id;
        this.page.current = 1;
        this.getTableData();
        this.setOpts();
    },
    methods: {
        processQueryForm () {
            let queryForm = {
            };
            queryForm.page = this.page.current - 1;
            queryForm.size = this.page.size;
            queryForm.fid = this.optForm.fid;
            return queryForm;
        },
        getTableData () {
            let _self = this;
            util.ajax.get('/ifc/list', { params: this.processQueryForm() }).then(res => {
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
        goIfcSave (row) {
            this.showModalIfcSave(row);
        },
        showModalIfcSave (row) {
	        if (row) {
	            this.optModal.ifcSave.title = '编辑接口';
	            let _self = this;
	            util.ajax.get('/ifc/detail?id=' + row.id).then(res => {
	                if (res.status === 200) {
	                    if (res.data.code === "0") {
	                        _self.prepareIfcSaveForm(res.data.content);
	                    }
	                }
	                this.optModal.ifcSave.loading = false;
	            }).catch(err => {
	                this.optModal.ifcSave.loading = false;
	                console.log(err);
	            });
	        } else {
	            this.clearIfcSaveForm ();
                this.optModal.ifcSave.loading = false;
	            this.optModal.ifcSave.title = '添加接口';
	        }
            this.optModal.ifcSave.show = true;
        },
        doIfcSave () {
            let _self = this;
            this.$refs.ifcSaveForm.validate((valid) => {
                if (valid) {
                    util.ajax.post('/ifc/save?fid=' + this.optForm.fid, this.processIfcSaveForm()).then(res => {
                        _self.getTableData();
                        _self.clearIfcSaveForm ();
                        _self.optModal.ifcSave.show = false;
                        _self.$Message.info(res.data.message);
                    }).catch(err => {
                        console.log(err);
                        _self.optModal.ifcSave.show = false;
                    });
                } else {
                    _self.optModal.ifcSave.okLoading = false;
                    _self.$nextTick(() => _self.optModal.ifcSave.okLoading = true);
                }
            });
        },
        prepareIfcSaveForm (data) {
            this.optForm.ifcSave.id = data.id;
            this.optForm.ifcSave.sn = data.sn.toString();
            this.optForm.ifcSave.name = data.name.toString();
            this.optForm.ifcSave.method = data.method.toString();
            this.optForm.ifcSave.dataProtocol = data.dataProtocol.toString();
            this.optForm.ifcSave.url = data.url.toString();
        },
        processIfcSaveForm () {
            let form = {
                id: this.optForm.ifcSave.id,
                sn: this.optForm.ifcSave.sn,
                name: this.optForm.ifcSave.name,
                method: this.optForm.ifcSave.method,
                dataProtocol: this.optForm.ifcSave.dataProtocol,
                url: this.optForm.ifcSave.url,
            };
            return form;
        },
        clearIfcSaveForm () {
            this.optForm.ifcSave.id = '';
            this.optForm.ifcSave.sn = '';
            this.optForm.ifcSave.name = '';
            this.optForm.ifcSave.method = '';
            this.optForm.ifcSave.dataProtocol = 'json';
            this.optForm.ifcSave.url = '';
            this.$refs.ifcSaveForm.resetFields();
        },
        cancelIfcSave () {
            this.$refs.ifcSaveForm.resetFields();
        },
        goIfcDelete (row) {
            let _self = this;
            this.$Modal.confirm({
                title: '删除',
                content: '确定要删除？',
                loading: true,
                onOk: () => {
                    let _modal = this.$Modal;
                    util.ajax.post('/ifc/delete?id=' + row.id).then(res => {
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
        goAttrReqDetail (row) {
            this.$router.push({
                name: 'attr_detail',
                params: {
                    id: row.id
                },
            });
        },
        goAttrResDetail (row) {
            this.$router.push({
                name: 'attr_detail',
                params: {
                    id: row.id
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
                                    this.goAttrReqDetail(params.row);
                                }
                            }
                        };
                    },
                    label: '请求属性'
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
                                    this.goAttrResDetail(params.row);
                                }
                            }
                        };
                    },
                    label: '应答属性'
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
                                    this.goIfcSave(params.row);
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
                                    this.goIfcDelete(params.row);
                                }
                            }
                        };
                    },
                    label: '删除'
                },
            ];
            if (opts.length > 0) {
	            this.columns.push({
	                title: '操作',
	                key: 'action',
	                fixed: 'right',
	                align: 'center',
	                render: (h, params) => {
	                    let actions = opts.map(optItem => {
	                        return h(optItem.widget, optItem.attrs(params), typeof optItem.label == 'function' ? optItem.label(params) : optItem.label);
	                        }
	                    );
	                    let count = actions.filter(action => action.data.style.display != "none").length;
	                    params.column.width = count * 70;
	                    return h('div', actions);
	                }
	            });
            }
        },
    },
};
</script>
