<template><div>
<Card>
	<Form :model="queryForm" :label-width="80">
		<Row>
			<Col span="6" style="margin-bottom: -15px;">
				<FormItem label="标签" prop="label">
					<Input type="text" v-model="queryForm.label" ></Input>
				</FormItem>
			</Col>
			<Col span="6" style="margin-bottom: -15px;">
				<FormItem label="名字" prop="name">
					<Input type="text" v-model="queryForm.name" ></Input>
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
<Modal width="800" v-model="genModal.show" @on-ok="genModal.show = false" title="生成代码">
<Row>
<Col>
    类型：<Select v-model="genModal.type" style="width:200px">
        <Option value="list-wsh" key="list-wsh">list-wsh</Option>
        <Option value="detail-wsh" key="detail-wsh">detail-wsh</Option>
        <Option value="list" key="list">list</Option>
    </Select>
    <Button type="primary" :loading="genModal.gening" @click="doGen">
        <span v-if="!genModal.gening">生成代码</span>
        <span v-else>生成中...</span>
    </Button>
	<Button type="primary" class="cbbtn" data-clipboard-target="#codeContainer" @click="$Message.info('已复制到剪切板')">复制代码</Button>
</Col>
<Col>
代码：<Input id="codeContainer" type="textarea" v-model="genModal.content" :rows="12"></Input>
    <Spin size="large" fix v-if="genModal.spinShow">生成中...</Spin>
</Col>
</Row>
</Modal>
<Modal width="700" v-model="saveModal.show" loading @on-ok="doSave" :title="saveModal.title">
        <Form :model="saveForm" :label-width="80">
		<Row>
			<Col span="10" style="margin-bottom: -15px;">
				<FormItem label="标签" prop="label">
					<Input type="text" v-model="saveForm.label" placeholder="请输入标签"></Input>
				</FormItem>
			</Col>
			<Col span="10" style="margin-bottom: -15px;">
				<FormItem label="名字" prop="name">
					<Input type="text" v-model="saveForm.name" placeholder="请输入名字"></Input>
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
import util from '@/libs/util'
import Clipboard from 'clipboard'

export default {
	data () {
        return {
        	genModal: {
        		show: false,
        		id: '',
        		content: '',
        		type: 'list-wsh',
        		spinShow: false,
        		gening: false
        	},
        	saveModal: {
        		show: false,
        		title: ''
        	},
        	queryForm: {
        		label: '',
        		name: ''
        	},
        	saveForm: {
        		id: '',
        		label: '',
        		name: ''
        	},
        	columnsList:[
				{
					title: '标签',
					key: 'label',
					align: 'center'
				},
				{
					title: '名字',
					key: 'name',
					align: 'center'
				},
				{
                    title: '操作',
                    key: 'action',
                    fixed: 'right',
                    width: 260,
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
                              			this.showModalGen(params.row.id)
                               		}
                           		}
                            }, '生成'),
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
                              			let args = {
                              				fid: params.row.id
                              			}
                                   		//util.openNewPage(this, 'order_info', argu);
                                   		this.$router.push({
                                       		name: 'attr_management',
                                       		params: args
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
    	const clipboard = new Clipboard('.cbbtn');
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
			this.queryForm = {
				label: '',
			    name: ''
			}
			this.doQuery()
    	},
    	doGen (id) {
    		this.genModal.content = ''
    		this.genModal.gening = true
    		this.genModal.spinShow = true
    		let _self = this
    		util.ajax.post('/gen/vue/' + this.genModal.id + '?type=' + this.genModal.type).then(res => {
    			if (res.status === 200) {
    				if (res.data.code === '0') {
    					_self.genModal.content = res.data.data;
    					_self.genModal.gening = false
    					_self.genModal.spinShow = false
    				}
    			}
    		}).catch(err => {
    			_self.genModal.gening = false
    			_self.genModal.spinShow = false
    			_self.$Message.error("类型无效")
    			console.log(err)
    		})
    	},
		doSave () {
			let _self = this
			util.ajax.post('/func/save', this.saveForm).then(function (res) {
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
					util.ajax.post('/func/delete/' + id).then(function (res) {
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
			util.ajax.get('/func/list?page=' + this.page.num + '&size=' + this.page.size).then(function (res) {
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
		showModalGen (id) {
			this.genModal.show = true
			this.genModal.id = id
			this.genModal.content = ''
		},
		showModalAdd () {
			this.clearSaveForm()
			this.saveModal.title = '添加功能'
			this.saveModal.show = true
		},
		showModalUpdate(id) {
			let _self = this
			util.ajax.get('/func/detail?id=' + id).then(function (res) {
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
    		this.saveForm = {
    			id: '',
				label: '',
			    name: ''
			}
    	},
    	clearPage () {
    		this.page.current = 1
    	},
		changePage (number) {
			this.page.num = number - 1
			this.getList()
		}
	}
}
</script>