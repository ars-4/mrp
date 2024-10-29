<template>

    <div class="parent">

        <h3>Bill Of Materials</h3>&nbsp;
        <span>
            <label class="btn btn-primary" for="bomFileReader">
                <span v-if="fileSelected">Uploaded</span>
                <span v-else>Upload</span>
            </label>
            <input type="file" id="bomFileReader" name="file" @change="onFileChange" hidden>
        </span>

    </div>

</template>

<style scoped>
.parent {
    display: flex;
    align-items: center;
    justify-content: space-between;
}
</style>

<script>

export default {

    name: 'BillOfMaterials',

    data() {
        return {
            saleForecast: false,
            fileSelected: false
        }
    },
    methods: {

        onFileChange: function (e) {
            const file = e.target.files[0];
            if (file) {
                this.fileSelected = true;
                const reader = new FileReader();
                reader.onload = (event) => {
                    let fileContent = event.target.result;
                    this.process_bom(fileContent);
                };
                reader.readAsText(file);
            }
        },

        process_bom: function (fileContent) {
            let lines = fileContent.split('\n');
            lines.pop();
            lines.forEach((line, i) => {
                lines[i] = line.replace("\r", "").split(',');
            })
            let transposed = lines[0].map((col, i) => lines.map(row => row[i]));
            let bomItems = [];
            for (let i = 0; i < transposed.length - 1; i++) {
                let productName = transposed[i][0];
                let productPrice = transposed[i + 2][0];
                let items = transposed[i].slice(1).filter(item => item !== '')
                let itemQtys = transposed[i + 1].slice(1).filter(item => item !== '')
                let itemPrices = transposed[i + 2].slice(1).filter(item => item !== '')

                let obj = {
                    "product": productName,
                    "price": Number(productPrice),
                    "items": [],
                }
                items.forEach((item, j) => {
                    obj.items.push({
                        "item": item,
                        "qty": Number(itemQtys[j]),
                        "price": Number(itemPrices[j]),
                    })
                })

                bomItems.push(obj);

                i = i + 2;
            }
            this.$emit('processed_bom', bomItems);
        },

    },

}

</script>