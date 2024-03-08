<template>
	<div class="container-fluid">
		<div class="row">
			<main role="main" class="col-md-12 ml-sm-auto col-lg-12 pt-3 px-4">
				<div class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pb-2 mb-3 ">
					<h2>VueSheet List</h2>
					<select v-model="currentSheetType" @change="changeSheetType">
						<option v-for="(type, index) in types" :value="index" :key="type">{{ type }}</option>
					</select>
					<div class="btn-toolbar mb-2 mb-md-0">
						<a href="https://docs.google.com/spreadsheets/d/12qEH-C0_ajN-Lch4_Pa0vN9IDfZgyZbJqy7abu8gvW4/edit?usp=sharing"
							class="btn btn-sm btn-outline-secondary" target="_blank">
							View Google Sheet
						</a>
					</div>
				</div>
				<div class="table-responsive">
					<table class="table table-striped ">
						<thead>
							<tr>
								<th>作品類型</th>
								<th>作品名稱</th>
								<th>集數</th>
								<th>完結</th>
							</tr>
						</thead>

						<tbody>
							<Row v-bind:key="row.id" v-for="row in rows" v-bind:row="row" @edit="openEditModal" />
							<EditModal ref="editModal" :row="rowToEdit" @close="closeEditModal" />
						</tbody>
					</table>
				</div>
			</main>
		</div>
	</div>
</template>

<script>
import Row from '@/components/Row.vue';
import EditModal from '@/components/EditModal.vue';
import bootstrap from 'bootstrap/dist/js/bootstrap.bundle.min.js';
const { GoogleSpreadsheet } = require('google-spreadsheet');
const creds = require('@/client_secret.json');

export default {
	name: "Sheet",
	components: {
		Row,
		EditModal // 確保引入模態窗口組件
	},
	data() {
		return {
			rows: [],
			loading: true,
			currentSheetType: 0, // You can default to a specific sheet
			types: ['電影', '動畫', '漫畫', '小說', '遊戲'],
			rowToEdit: null,
			myModal: '',
		}
	},
	methods: {
		async accessSpreadSheet() {
			const doc = new GoogleSpreadsheet('12qEH-C0_ajN-Lch4_Pa0vN9IDfZgyZbJqy7abu8gvW4');
			await doc.useServiceAccountAuth(creds);
			await doc.loadInfo();
			const sheet = doc.sheetsByIndex[this.currentSheetType]; // Use currentSheetType to select the sheet
			const rows = await sheet.getRows({ offset: 1 });
			this.rows = rows;
		},
		changeSheetType() {
			this.accessSpreadSheet(); // Reload the data
		},
		openEditModal(row) {
			this.rowToEdit = row;
			if (!this.myModal) {
				this.myModal = new bootstrap.Modal(document.getElementById("modalEdit"));
			}
			this.myModal.show();
		},
		closeEditModal() {
			if (this.myModal) {
				this.myModal.hide();
			}
		},
	},
	created() {
		this.accessSpreadSheet();
	}
}
</script>