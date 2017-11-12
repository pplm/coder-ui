<template><div>
<Card>
	<Form :model="formItem" :label-width="80">
		<Row>
			<Col span="6" style="margin-bottom: -15px;">
				<FormItem label="会员姓名" prop="name">
					<Input type="text" ></Input>
				</FormItem>
			</Col>
			<Col span="6" style="margin-bottom: -15px;">
				<FormItem label="手机号码" prop="name">
					<Input type="text" ></Input>
				</FormItem>
			</Col>
			<Col span="7" style="text-align: center; margin-bottom: -15px;">
				<FormItem>
					<Button type="primary" icon="android-search">查询</Button>
					<Button type="error" icon="android-refresh">清空</Button>
					<Button type="success" icon="archive">导出</Button>
				</FormItem>
			</Col>
		</Row>
	</Form>
</Card>
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
export default {
	data () {
        return {
        	formItem: {

        	},
        	columnsList:[
        		{
                    title: '订单单号',
                    key: 'order_no',
                    align: 'center'
                },
                {
                    title: '会员姓名',
                    key: 'name',
                    align: 'center'
                },
                {
                	title: '手机号码',
                    key: 'mobile',
                    align: 'center'
                },
                {
                	title: '充值金额',
                    key: 'money',
                    align: 'center'
                },
                {
                	title: '金币数',
                    key: 'coin',
                    align: 'center'
                },

        	],
        	tableData: [
        		{
        			order_no: 1,
        			name: "测试",
        			mobile: 123,
        			money: 111,
        			coin: 100
        		},
        		{
        			order_no: 2,
        			name: "测试2",
        			mobile: 123123,
        			money: 1111,
        			coin: 200
        		},
        	],
        	page: {
        		total: 100,
        		size: 10,
        		current: 1
        	}
        }
    },
    methods: {
		changePage (number) {
			console.log(number)
		},
	}
}
</script>