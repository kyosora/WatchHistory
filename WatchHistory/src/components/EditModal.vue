<template>
    <div class="modal fade" id="modalEdit" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel"
        aria-hidden="true">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="exampleModalLabel">Edit Row</h5>
                    <button type="button" class="close" @click="$emit('close')" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <div class="modal-body">
                    <form @submit.prevent="submitEdit">
                        <div class="mb-3">
                            <label class="form-label">作品名稱</label>
                            <input type="text" class="form-control" v-model="editedRow.作品名稱">
                        </div>
                        <div class="mb-3">
                            <label class="form-label">集數</label>
                            <input type="text" class="form-control" v-model="editedRow.集數">
                        </div>
                        <div class="mb-3 form-check">
                            <input type="checkbox" class="form-check-input" id="completedCheck" v-model="editedRow.完結">
                            <label class="form-check-label" for="completedCheck">是否完結</label>
                        </div>
                        <!-- 其他字段... -->
                        <div class="modal-footer">
                            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">取消</button>
                            <button type="submit" class="btn btn-primary">保存</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</template>
  
<script>
import { GoogleSpreadsheet } from 'google-spreadsheet';
import bootstrap from 'bootstrap/dist/js/bootstrap.bundle.min.js';
const creds = require('@/client_secret.json');
export default {
    name: "EditModal",
    props: ["row"],
    data() {
        return {
            editedRow: { ...this.row } // 創建行數據的副本以供編輯
        };
    },
    methods: {
			getSheetIndexByType(type) {
			const types = ['電影', '動畫', '漫畫', '小說', '遊戲'];
			return types.indexOf(type);
		},
        async submitEdit() {
            console.log(this.row);
            console.log(this.editedRow);
            // 加載文檔和工作表
            const doc = new GoogleSpreadsheet('12qEH-C0_ajN-Lch4_Pa0vN9IDfZgyZbJqy7abu8gvW4');
            await doc.useServiceAccountAuth(creds);
            await doc.loadInfo();
			const sheetIndex = this.getSheetIndexByType(this.editedRow.作品類型); // Get the index based on selected type
			const sheet = doc.sheetsByIndex[sheetIndex];
			console.log(sheetIndex);
            // 根據您的行ID或其他標識符找到行
            const rows = await sheet.getRows();
            console.log()
            const rowToUpdate = rows.find(r => r.作品名稱 === this.row.作品名稱); // 請確保您有某種方式來標識要更新的行
            // 更新字段
            rowToUpdate.作品名稱 = this.editedRow.作品名稱;
            rowToUpdate.集數 = this.editedRow.集數;
            // 如果你需要將布爾值轉換為其他格式，例如字符串
            this.editedRow.完結 = this.editedRow.完結 ? '是' : '否';
            rowToUpdate.完結 = this.editedRow.完結;
            // 保存更新
			console.log(rowToUpdate);
            await rowToUpdate.save();

            // 通知父組件完成，以便可能的後續操作，例如刷新列表
            this.$emit('updated', this.editedRow);

            // 關閉模態窗口
            var myModalEl = document.getElementById('modalEdit');
            var myModal = bootstrap.Modal.getInstance(myModalEl);
            myModal.hide();
            // 調用父組件方法重新拉取數據
            this.$parent.accessSpreadSheet();
        },
    },
    watch: {
        row: {
            handler(newValue) {
                if (newValue) {
                    this.editedRow = { ...newValue, 完結: newValue.完結 === '是' };
                }
            },
            immediate: true
        }
    }
}
</script>