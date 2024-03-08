<template>
	<tr>
		<td>{{row.作品類型}}</td>
		<td>{{row.作品名稱}}</td>
		<td>{{row.集數}}</td>
		<td>{{row.完結}}</td>
		<td>
			<button @click="editRow">Edit</button>
		</td>
	</tr>
</template>

<script>
	export default {
		name: "Row",
		props: ["row"],
		methods: {
			editRow() {
				debugger
				this.$emit('edit', this.row);
			},
		},
	}
</script>