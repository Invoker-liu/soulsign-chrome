<template>
	<mu-dialog
		v-loading="loading"
		title="脚本推荐"
		:width="360"
		:open="Boolean(open)"
		class="pages-lists"
		scrollable
		@update:open="close"
	>
		<ul>
			<li v-for="(item, i) in list" :key="i" class="tac">
				<a
					:href="'#https://gitee.com/inu1255/soulsign-chrome/raw/master/public/demos/' + item[1]"
					>{{ item[0] }}</a
				>
			</li>
		</ul>
		<mu-button slot="actions" flat @click="close">关闭</mu-button>
	</mu-dialog>
</template>
<script>
import utils from "../../common/client";

export default {
	props: {
		open: {},
	},
	data() {
		return {
			loading: false,
			list: [],
			sort: {name: "name", order: "asc"},
		};
	},
	watch: {
		open() {
			this.refresh();
		},
	},
	mounted() {
		if (this.open) this.refresh();
	},
	methods: {
		refresh() {
			if (!this.open) return;
			this.$with(async () => {
				let {data} = await utils.axios.get(
					`https://gitee.com/inu1255/soulsign-chrome/raw/master/public/demos.json`
				);
				this.list = data;
			});
		},
		close() {
			this.$emit("update:open", false);
		},
	},
};
</script>
<style lang="less">
.pages-lists {
	ul {
		padding: 0;
		margin: 0;
		min-height: 40vh;
	}
	li {
		list-style: none;
		margin: 3px 5px;
	}
}
</style>
