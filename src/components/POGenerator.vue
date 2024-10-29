<template>
    <h1>PO Generator</h1>

    <div class="section">
        <h3>Items</h3>
        <table>
            <thead>
                <tr>
                    <th>Item</th>
                    <th>Qty</th>
                    <th>Price</th>
                    <th>Safety Stock</th>
                    <th>Lead Time</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="item in itemsMonthlyConsumption" :key="item">
                    <td>{{ item.item }}</td>
                    <td>{{ item.consumption }}</td>
                    <td>{{ item.price }}</td>
                    <td>{{ item.safety }}</td>
                    <td><input type="number" v-model="item.lt"></td>
                    <td><input type="number" v-model="item.closing"></td>
                </tr>
            </tbody>
        </table>
        <button class="btn btn-primary" style="width:100%" @click="generate_po">Generate</button>
    </div>

</template>

<style scoped>
table input {
    width: 100px;
    padding: 6px;
    border-radius: 4px;
    outline: none;
    border: 1px solid #c3c3c3;
}

table,
table td,
table th {
    border: 1px solid #242424;
    border-collapse: collapse;
}

table td,
table th {
    padding: 8px;
    text-align: center;
    width: 100px;
}
</style>

<script>

export default {

    name: 'POGenerator',
    props: {
        consumption: Array
    },

    data() {
        return {
            itemsMonthlyConsumption: []
        }
    },

    methods: {

        setter: function () {
            this.itemsMonthlyConsumption = this.consumption[0].items.map(item => ({
                item: item.item,
                consumptionList: [],
                price: item.price
            }));

            this.consumption.forEach(day => {
                day.items.forEach(dayItem => {
                    const targetItem = this.itemsMonthlyConsumption.find(i => i.item === dayItem.item);
                    if (targetItem) {
                        targetItem.consumptionList.push(dayItem.consumption);
                    }
                });
            });

            this.itemsMonthlyConsumption.forEach(item => {
                item.consumption = item.consumptionList.reduce((a, b) => a + b, 0);
                item.maxC = Math.max(...item.consumptionList);
                item.avgC = item.consumption / item.consumptionList.length;
            });

            console.log(this.itemsMonthlyConsumption)
        },

        updateSafetyStock(item) {
            const newSafetyStock = Math.round((item.maxC - item.avgC) * item.lt);
            item.safety = newSafetyStock;
            item.reorder = (item.avgC * item.lt) + newSafetyStock;
            item.optimum = item.reorder + item.safety;
        },

        generate_po: function () {
            let month = new Date();
            month.setMonth(month.getMonth() + 1);
            let daysLength = new Date(month.getFullYear(), month.getMonth() + 1, 0).getDate();
            let purchases = [];
            let closing = [];
            for (let i = 0; i < this.itemsMonthlyConsumption.length; i++) {
                let itemQty = this.itemsMonthlyConsumption[i].closing;
                let days = []
                for (let j = 0; j < this.itemsMonthlyConsumption[i].consumptionList.length; j++) {
                    itemQty -= this.itemsMonthlyConsumption[i].consumptionList[j];
                    if (itemQty <= this.itemsMonthlyConsumption[i].safety) {
                        if (this.itemsMonthlyConsumption[i].lt + j + 1 <= daysLength) {
                            itemQty += (this.itemsMonthlyConsumption[i].optimum - itemQty);
                        }
                        days.push({
                            day: j + 1,
                            qty: itemQty,
                            price: this.itemsMonthlyConsumption[i].price * itemQty,
                            arrival: this.itemsMonthlyConsumption[i].lt + j + 1
                        });
                        if (this.itemsMonthlyConsumption[i].lt + j + 1 >= daysLength) {
                            closing.push({
                                item: this.itemsMonthlyConsumption[i].item,
                                qty: itemQty
                            })
                        }
                    }
                }
                purchases.push({
                    item: this.itemsMonthlyConsumption[i].item,
                    days: days
                })
            }
            console.log(closing)

            let csvContent = "data:text/csv;charset=utf-8,";
            purchases.forEach(function (rowArray) {
                let row = rowArray.item + "\n" + rowArray.days.map(function (d) {
                    return d.day + "," + d.qty + "," + d.price + "," + d.arrival;
                }).join("\n");
                csvContent += row + "\n\n";
            });
            var encodedUri = encodeURI(csvContent);
            var link = document.createElement("a");
            link.setAttribute("href", encodedUri);
            link.setAttribute("download", "po.csv");
            document.body.appendChild(link);
            link.click();


        }


    },

    watch: {
        itemsMonthlyConsumption: {
            handler(newValue) {
                newValue.forEach(item => {
                    this.updateSafetyStock(item);
                });
            },
            deep: true
        }
    },

    mounted() {
        this.itemsMonthlyConsumption.forEach(item => {
            this.$watch(
                () => item.lt,
                () => this.updateSafetyStock(item)
            );
        });
    },

    beforeMount() {
        this.setter()
    }

}

</script>