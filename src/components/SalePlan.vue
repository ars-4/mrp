<template>

    <div class="plan">

        <h3>Sale Plan</h3>&nbsp;
        <span>
            <label class="btn btn-primary" for="saleFileReader">
                <span v-if="fileSelected">Uploaded</span>
                <span v-else>Upload</span>
            </label>
            <input type="file" id="saleFileReader" name="file" @change="onFileChange" hidden>
        </span>
        <span>
            <input type="checkbox" id="saleForecast" v-model="saleForecast">
            <label for="saleForecast">Sale Forecast</label>
        </span>

    </div>

</template>

<style scoped>

.plan {
    display: flex;
    align-items: center;
    justify-content: space-between;
}

</style>

<script>

export default {

    name: 'SalePlan',

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
                    this.saleForecast ? this.saleWithForecast(fileContent) : this.saleWithoutForecast(fileContent);
                };
                reader.readAsText(file);
            }
        },

        saleWithoutForecast: function (fileContent) {
            let lines = fileContent.split('\n');
            lines.pop();
            lines.forEach((line, i) => {
                lines[i] = line.replace("\r", "").split(',');
            })
            let transposed = lines[0].map((col, i) => lines.map(row => row[i]));
            let days = transposed[0].slice(1).map(day => Number(day));
            let sales = transposed[1].slice(1).map(sale => Number(sale));
            let salesObject = [];
            for (let i = 0; i < days.length; i++) {
                salesObject.push({ day: days[i], sale: sales[i] });
            }
            this.$emit('processed_sales', salesObject);
        },

        saleWithForecast: function (fileContent) {
            let lines = fileContent.split('\n');
            lines.pop();
            lines.forEach((line, i) => {
                lines[i] = line.replace("\r", "").split(',');
            })
            let transposed = lines[0].map((col, i) => lines.map(row => row[i]));
            let months = transposed[0].slice(1).map(day => Number(day)).filter(month => month !== 0);
            let monthlySales = transposed[1].slice(1).map(sale => Number(sale)).filter(month => month !== 0);
            let days = transposed[2].slice(1).map(sale => Number(sale));
            let dailySales = transposed[3].slice(1).map(sale => Number(sale));
            console.log(months, monthlySales, days, dailySales);
        },

    },

}

</script>