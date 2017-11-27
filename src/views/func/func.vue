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
<Modal width="900" v-model="genModal.show" @on-ok="genModal.show = false" title="生成代码">
    <Row>
        <Col style="margin-bottom: 5px;">
      类型：<Select v-model="genModal.type" style="width:200px">
                <Option value="list-wsh" key="list-wsh">list-wsh</Option>
                <Option value="list" key="list">list</Option>
                <Option value="detail" key="detail">detail</Option>
                <Option value="router-item" key="router-item">router-item</Option>
            </Select>
            <Button type="primary" :loading="genModal.gening" @click="doGen">
                <span v-if="!genModal.gening">生成代码</span>
                <span v-else>生成中...</span>
            </Button>
	        <Button type="primary" class="cbbtn" data-clipboard-target="#codeContainer" @click="$Message.info('已复制到剪切板')">复制代码</Button>
        </Col>
        <Col>
            <Input id="codeContainer" type="textarea" v-model="genModal.content" :rows="15"></Input>
            <Spin size="large" fix v-if="genModal.spinShow">生成中...</Spin>
        </Col>
    </Row>
</Modal>
<Modal width="800" v-model="saveModal.show" loading @on-ok="doSave" :title="saveModal.title">
        <Form :model="saveForm" :label-width="80">
		<Row>
			<Col span="8" style="margin-bottom: -15px;">
				<FormItem label="标签" prop="label">
					<Input type="text" v-model="saveForm.label" placeholder="请输入标签"></Input>
				</FormItem>
			</Col>
			<Col span="8" style="margin-bottom: -15px;">
				<FormItem label="名字" prop="name">
					<Input type="text" v-model="saveForm.name" placeholder="请输入名字"></Input>
				</FormItem>
			</Col>
			<Col span="8" style="margin-bottom: -15px;">
				<FormItem label="关联属性" prop="relaAttr">
					<Input type="text" v-model="saveForm.relaAttr" placeholder="请输入关联属性"></Input>
				</FormItem>
			</Col>
      <Col span="8" style="margin-bottom: -15px;">
        <FormItem label="权限值" prop="permissionTag">
          <Input type="text" v-model="saveForm.permissionTag" placeholder="请输入权限值"></Input>
        </FormItem>
      </Col>
			<Col span="12" style="margin-bottom: -15px;">
				<FormItem label="备注" prop="textarea">
					<Input type="textarea" v-model="saveForm.remark" placeholder="请输入备注"></Input>
				</FormItem>
			</Col>
		</Row>
		</Form>
</Modal>
<Card>
<Button type="primary" icon="plus" style="margin-bottom: 5px; margin-top: -10px; text-align:right;" @click="showModalAdd">添加</Button>
	<Table :columns="columns" :data="tableData" border></Table>
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
        		name: '',
        		relaName: ''
        	},
        	saveForm: {
        		id: '',
        		label: '',
        		name: '',
        		relaName: '',
            permissionTag: '',
        		remark: ''
        	},
        	columns:[
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
          title: '关联属性',
					key: 'relaAttr',
					align: 'center'
				},
        {
          title: '权限值',
          key: 'permissionTag',
          align: 'center'
        },
				{
          title: '备注',
					key: 'remark',
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
                              		icon: 'gear-a',
                              		size: 'small'
                                },
                                style: {
          							marginRight: '5px'
        						},
                                on: {
                              		click: () => {
                                   		this.$router.push({
                                       		name: 'opt_management',
                                       		params: {
                                       			fid: params.row.id
                                       		}
                                   		})
                               		}
                           		}
                            }, '操作项'),
                      		h('Button', {
                                props: {
                                    type: 'primary',
                              		icon: 'ios-list-outline',
                              		size: 'small'
                                },
                                style: {
          							marginRight: '5px'
        						},
                                on: {
                              		click: () => {
                                   		this.$router.push({
                                       		name: 'attr_management',
                                       		params: {
                                       			fid: params.row.id
                                       		}
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
        	}
        }
    },
    mounted () {
      console.log("11111")
    	const clipboard = new Clipboard('.cbbtn')
      console.log("22222")
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
    					_self.genModal.content = res.data.content;
    					_self.genModal.gening = false
    					_self.genModal.spinShow = false
    				}
    			}
    		}).catch(err => {
    			_self.genModal.gening = false
    			_self.genModal.spinShow = false
    			_self.$Message.error(res.data.message)
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
			util.ajax.get('/func/list?page=' + (this.page.current - 1) + '&size=' + this.page.size).then(function (res) {
				if (res.status === 200) {
					if (res.data.code === "0") {
						_self.tableData = res.data.content.content
						_self.page.total = res.data.content.totalElements
						_self.page.size = res.data.content.size
						_self.page.current = res.data.content.number + 1
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
						_self.saveForm = res.data.content
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
			    name: '',
          permissionTag: '',
          remark: ''
			}
    	},
    	clearPage () {
    		this.page.current = 1
    	},
		changePage (number) {
			this.page.current = number
			this.getList()
		}
	}
}
</script>