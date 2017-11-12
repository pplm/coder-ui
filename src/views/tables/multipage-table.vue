<template>
    <div>
        <Table :ref="refs" :columns="columnsList" :data="thisTableData" border></Table>
		<div style="margin: 10px;overflow: hidden">
            <div style="float: right;">
                <Page :total="100" :current="1" @on-change="changePage"></Page>
            </div>
        </div>
    </div>
</template>
<script>
	export default {
		name: 'multipageTable',
		props: {
			refs: String,
			columnsList: Array,
			tableData: Array,
		},
		data () {
			return {
				columns: [],
				thisTableData: []
			};
		},
		created () {
			this.init();
		},
		methods: {
			init () {
			},
            changePage () {
            }
		},
		watch: {
			tableData (data) {
				this.init();
			}
		}
	}
</script>