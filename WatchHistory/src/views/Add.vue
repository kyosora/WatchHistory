<template>
	<div class="container">
		<div class="row">
			<div class="col-lg-12 mt-4">
				<div class="d-flex justify-content-between flex-wrap flex-md-nowrap align-items-center pb-2">
					<h4 class="mb-3">New row</h4>
					<div class="btn-toolbar mb-2 mb-md-0">
						<router-link to="/" class="btn btn-sm mr-1 btn-outline-secondary">
							Back
						</router-link>
						<a href="https://docs.google.com/spreadsheets/d/1oLGI1u68sPh-P1yhWaU7j_WUTHjFMY1aUu6uINfTb0Q/edit?usp=sharing"
							class="btn btn-sm btn-outline-secondary" target="_blank">
							View Google Sheet
						</a>
					</div>
				</div>
				<div v-if="showMsg" class="alert alert-success alert-dismissible fade show" role="alert">
					{{ message }}
					<button type="button" class="close" data-dismiss="alert" aria-label="Close">
						<span aria-hidden="true">&times;</span>
					</button>
				</div>
				<form @submit.prevent="addRow" class="form-group">
					<div class="row">
						<div class="col-md-6 mb-3">
							<label for="selectedType">作品類型</label>
							<select v-model="selectedType" class="form-control">
								<option disabled value="">請選擇作品類型</option>
								<option>電影</option>
								<option>動畫</option>
								<option>漫畫</option>
								<option>小說</option>
								<option>遊戲</option>
							</select>
						</div>
						<div class="col-md-6 mb-3">
							<label for="workName">作品名稱</label>
							<input type="text" v-model="workName" placeholder="作品名稱" class="form-control" />
						</div>
						<div class="col-md-6 mb-3">
							<label for="episode">集數</label>
							<input type="number" v-model="episode" placeholder="集數" class="form-control" />
						</div>
						<div class="col-md-2 mb-3">
							<label for="isFinished">是否完結</label>
							<input type="checkbox" v-model="isFinished" class="form-check" />
						</div>
						<div class="col-md-6">
							<button class="btn btn-primary btn-block mt-4" type="submit">新增紀錄</button>
						</div>
					</div>
				</form>
			</div>
		</div>
		<div class="row mt-4">
			<div class="col-lg-12">
				<h4 class="mb-3">大量匯入</h4>
				<div class="form-group">
					<label for="importType">作品類型</label>
					<select v-model="importType" class="form-control mb-3">
						<option disabled value="">請選擇作品類型</option>
						<option>電影</option>
						<option>動畫</option>
						<option>漫畫</option>
						<option>小說</option>
						<option>遊戲</option>
					</select>
					<input type="file" @change="handleFileUpload" class="form-control-file" />
					<button @click="importRecords" class="btn btn-primary btn-block mt-4">匯入紀錄</button>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
const { GoogleSpreadsheet } = require('google-spreadsheet');
const creds = require('@/client_secret.json');
export default {
	name: "AddRow",
	data() {
		return {
			selectedType: '',
			workName: '',
			episode: null,
			isFinished: false,
			showMsg: false,
			message: '',
			file: null,
			importType: '',
		}
	},
	methods: {
		async addRow() {
			const newRow = {
				'作品類型': this.selectedType,
				'作品名稱': this.workName,
				'集數': this.episode,
				'完結': this.isFinished ? '是' : '否',
			}

			const doc = new GoogleSpreadsheet('12qEH-C0_ajN-Lch4_Pa0vN9IDfZgyZbJqy7abu8gvW4');
			await doc.useServiceAccountAuth(creds);
			await doc.loadInfo();

			const sheetIndex = this.getSheetIndexByType(this.selectedType); // Get the index based on selected type
			const sheet = doc.sheetsByIndex[sheetIndex];
			await sheet.addRow(newRow);

			this.message = "New row added!";
			this.showMsg = true;
		},
		getSheetIndexByType(type) {
			const types = ['電影', '動畫', '漫畫', '小說', '遊戲'];
			return types.indexOf(type);
		},
		handleFileUpload(event) {
			this.file = event.target.files[0];
		},
		importRecords() {
			if (!this.importType) {
				this.message = "請先選擇作品類型!";
				this.showMsg = true;
				return;
			}

			const reader = new FileReader();
			reader.onload = async (event) => {
				const text = event.target.result;
				const lines = text.split('\n').filter(line => line && line.trim().length > 0); // 過濾掉空行或未定義的行
				const doc = new GoogleSpreadsheet('12qEH-C0_ajN-Lch4_Pa0vN9IDfZgyZbJqy7abu8gvW4');
				await doc.useServiceAccountAuth(creds);
				await doc.loadInfo();

				const sheetIndex = this.getSheetIndexByType(this.importType);
				const sheet = doc.sheetsByIndex[sheetIndex];
				const rows = await sheet.getRows(); // 取得已有的作品名稱

				const newRows = [];
				for (const line of lines) {
					// 根據你的文本格式解析每一行
					let parts = line.split('. ')[1]; // 假設用句號分隔
					console.log(parts)
					parts = parts.split('（');
					let isEnd = '否'
					if (parts[1]) {
						if (parts[1].indexOf('完結') != -1) {
							isEnd = '是';
						} else {
							isEnd = '否';
						}
					} else {
						isEnd = '否';
					}
					debugger
					// 檢查作品名稱是否已存在
					if (!rows.find(row => row['作品名稱'] === parts[0])) {
						newRows.push({
							'作品類型': this.importType,
							'作品名稱': parts[0],
							'集數': parts[1] ? parts[1].replace(/[^\d]/g, '') : '',
							'完結': isEnd
						});
					}
				}
				debugger
				// 分批次處理
				const batchSize = 60;
				for (let i = 0; i < newRows.length; i += batchSize) {
					await sheet.addRows(newRows.slice(i, i + batchSize)); // 批次添加
					if (i + batchSize < newRows.length) {
						await new Promise(resolve => setTimeout(resolve, 1000)); // 每次批次之間等待1秒
					}
				}

				this.message = "紀錄已成功匯入!";
				this.showMsg = true;
			};
			reader.readAsText(this.file);
		}

	}
}
</script>

<style scoped></style>
