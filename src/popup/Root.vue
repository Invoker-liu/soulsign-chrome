<template>
	<div class="root">
		<div class="tar">
			<a @click="reload">重启</a>
			<a @click="tab = 0">设置</a>
			<a @click="tab = 1">辅助</a>
			<a target="_blank" href="/options.html">任务管理</a>
		</div>
		<div v-if="tab == 0">
			<label>
				<span>掉线通知频率(秒)</span>
				<input v-model="config.notify_freq" type="number" /> </label
			><br />
			<label>
				<span>失败重试频率(秒)</span>
				<input v-model="config.retry_freq" type="number" /> </label
			><br />
			<label>
				<span>任务循环频率(秒)</span>
				<input v-model="config.loop_freq" type="number" /> </label
			><br />
			<label>
				<span>签到开始时间(秒),默认8点整</span>
				<input v-model="config.begin_at" type="number" /> </label
			><br />
			<label>
				<span>自动更新脚本</span>
				<input v-model="config.upgrade" type="checkbox" /> </label
			><br />
			<label>
				<span>自动更新频率(秒),默认1天</span>
				<input v-model="config.upgrade_freq" type="number" /> </label
			><br />
			<label>
				<span>任务超时限制(秒),默认60秒</span>
				<input v-model="config.timeout" type="number" /> </label
			><br />
			<label>
				<span>开启捐赠,京东购物为作者提供收益</span>
				<input v-model="config.donate" type="checkbox" /> </label
			><br />
			<div class="tar">
				<button @click="save">保存</button>
			</div>
		</div>
		<div v-else>
			<label>
				<span>cookie</span>
				<textarea v-model="cookie" cols="30" rows="10"></textarea></label
			><br />
			<div class="tar">
				<button @click="saveCookie((cookie = ''))">清空</button>
				<button @click="copyCookie">复制</button>
				<button @click="saveCookie">保存</button>
			</div>
		</div>
	</div>
</template>
<script>
import Vue from "vue";
import utils from "../common/client";

export default {
	data: function () {
		return {
			config: {},
			tab: 0,
			cookie: "",
		};
	},
	async mounted() {
		this.config = await utils.request("config/get");
		chrome.tabs.query({active: true, currentWindow: true}, (tabs) => {
			if (!tabs.length) return;
			let url = tabs[0].url;
			chrome.cookies.getAll(
				{url},
				(cookies) => (this.cookie = cookies.map((x) => `${x.name}=${x.value}`).join(";\n"))
			);
		});
	},
	methods: {
		save() {
			utils.request("config/set", this.config);
		},
		copyCookie() {
			utils.copy(this.cookie);
		},
		saveCookie() {
			chrome.tabs.query({active: true, currentWindow: true}, (tabs) => {
				if (!tabs.length) return;
				let url = tabs[0].url;
				console.log(url);
				chrome.cookies.getAll({url}, (cookies) => {
					Promise.all(
						cookies.map(
							(x) =>
								new Promise((resolve, reject) => {
									chrome.cookies.remove({url, name: x.name}, resolve);
								})
						)
					).then(() => {
						this.cookie
							.split(/;\s*/)
							.filter((x) => x.trim())
							.map((x) => x.split("="))
							.map((x) => ({name: x[0].trim(), value: x[1].trim(), url}))
							.map((x) => chrome.cookies.set(x));
					});
				});
			});
		},
		reload() {
			chrome.runtime.reload();
		},
	},
};
</script>
<style lang="less">
.root {
	min-width: 240px;
	input {
		margin-bottom: 3px;
	}
	input[type="number"] {
		width: 56px;
	}
}
</style>
