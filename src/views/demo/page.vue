<template>
    <Page :total="total" :current="3" :page-size="50" :page-size-opts="[50, 100, 200, 400]" show-total show-sizer show-elevator @on-change="changePage" @on-page-size-change="changeSize">
		总计 {{total}} 条
	</Page>
</template>
<script>
    export default {
	data () {
        return {
            total: 1000
        };
    },
        methods: {
			changePage (number) {
				console.log(number)
			},
			changeSize (size) {
				console.log(size)
			}
		}
    }
</script>