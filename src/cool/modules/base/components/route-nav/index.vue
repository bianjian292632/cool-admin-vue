<template>
	<div class="cl-route-nav">
		<p v-if="browser.isMini" class="title">
			{{ lastName }}
		</p>

		<template v-else>
			<el-breadcrumb>
				<el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
				<el-breadcrumb-item v-for="(item, index) in list" :key="index">{{
					(item.meta && item.meta.label) || item.name
				}}</el-breadcrumb-item>
			</el-breadcrumb>
		</template>
	</div>
</template>

<script lang="ts">
import { computed, defineComponent, ref, watch } from "vue";
import _ from "lodash";
import { isEmpty } from "/@/core/utils";
import { useCool } from "/@/core";

export default defineComponent({
	name: "cl-route-nav",

	setup() {
		const { store, route } = useCool();

		// 数据列表
		const list = ref<any[]>([]);

		// 监听路由变化
		watch(
			() => route,
			(val: any) => {
				const deep = (item: any) => {
					if (route.path === "/") {
						return false;
					}

					if (item.path == route.path) {
						return item;
					} else {
						if (item.children) {
							const ret = item.children.map(deep).find(Boolean);

							if (ret) {
								return [item, ret];
							} else {
								return false;
							}
						} else {
							return false;
						}
					}
				};

				list.value = _(store.getters.menuGroup)
					.map(deep)
					.filter(Boolean)
					.flattenDeep()
					.value();

				if (isEmpty(list.value)) {
					list.value.push(val);
				}
			},
			{
				immediate: true
			}
		);

		// 最后一个节点名称
		const lastName = computed(() => _.last(list.value).name);

		const browser = computed(() => store.getters.browser);

		return {
			list,
			lastName,
			browser
		};
	}
});
</script>

<style lang="scss">
.cl-route-nav {
	white-space: nowrap;

	.el-breadcrumb {
		margin: 0 10px;

		&__inner {
			font-size: 13px;
			padding: 0 10px;
			font-weight: normal;
			letter-spacing: 1px;
		}
	}

	.title {
		font-size: 15px;
		font-weight: 500;
		margin-left: 5px;
	}
}
</style>
