<template>

    <div class="consumption">

        <h3>Consumption</h3>
        <span>
            <label><b>Sale</b></label>&nbsp;
            <input type="number" v-model="sale">
        </span>
        <br>
        <table>
            <thead>
                <tr>
                    <th>Product</th>
                    <th>Qty</th>
                    <th>Price</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="product in products" :key="product">
                    <td>{{ product.product }}</td>
                    <td><input type="number" v-model="product.qty"></td>
                    <td>{{ product.price }}</td>
                    <td>{{ product.price * product.qty }}</td>
                    <td>{{ (((product.price * product.qty) / sale) * 100).toFixed(2) }}%</td>
                </tr>
            </tbody>
        </table>
        <br>
        <button class="btn btn-primary" style="width:100%" @click="consume">Consume</button>

    </div>

</template>

<style scoped>
.consumption {
    display: flex;
    align-items: center;
    justify-content: space-between;
    flex-direction: column;
    border: 1px solid #c3c3c3;
    padding: 8px 12px;
    border-radius: 4px;
}

.consumption span input {
    width: 243px;
    padding: 6px;
    border-radius: 4px;
    outline: none;
    border: 1px solid #c3c3c3;
}

.consumption table input {
    width: 100px;
    padding: 6px;
    border-radius: 4px;
    outline: none;
    border: 1px solid #c3c3c3;
}

.consumption table,
.consumption table td,
.consumption table th {
    border: 1px solid #242424;
    border-collapse: collapse;
}

.consumption table td,
.consumption table th {
    padding: 8px;
    text-align: center;
    width: 100px;
}
</style>

<script>

export default {

    name: 'SalePlan',

    props: {
        products: Array,
        sales: Array
    },

    data() {
        return {
            saleForecast: false,
            fileSelected: false,
            sale: 0

        }
    },
    methods: {
        consume: function () {
            let raw_items = []
            this.products.forEach(product => {
                if (this.sale == 0) return
                product.ratio = Number(Number(product.qty) * Number(product.price)) / Number(this.sale)
                product.items.forEach(item => {
                    let existingItem = raw_items.find(rawItem => rawItem.item === item.item)
                    if (existingItem) {
                        existingItem.qty += Number(item.qty)
                        existingItem.price = Number(item.price)
                    } else {
                        raw_items.push({
                            item: item.item,
                            qty: Number(item.qty),
                            price: Number(item.price)
                        })
                    }
                })
            })

            this.future_item_consumption()
        },

        future_item_consumption: function () {
            let dailyData = [];
            let monthlyProductsConsumptionMap = {};
            this.sales.forEach(sale => {
                let saleDetails = {
                    day: sale.day,
                    sale: sale.sale,
                    products: []
                };
                this.products.forEach(product => {
                    let productConsumption = (product.ratio * sale.sale) / product.price;
                    if (!monthlyProductsConsumptionMap[product.product]) {
                        monthlyProductsConsumptionMap[product.product] = {
                            product: product.product,
                            totalConsumption: 0,
                            items: []
                        };
                    }
                    monthlyProductsConsumptionMap[product.product].totalConsumption += productConsumption;
                    product.items.forEach(item => {
                        let itemConsumption = item.qty * productConsumption;
                        let monthlyProductItems = monthlyProductsConsumptionMap[product.product].items;
                        let existingItem = monthlyProductItems.find(i => i.item === item.item);
                        if (existingItem) {
                            existingItem.consumption += itemConsumption;
                        } else {
                            monthlyProductItems.push({
                                item: item.item,
                                consumption: itemConsumption,
                                qty: item.qty,
                                price: item.price
                            });
                        }
                    });
                    saleDetails.products.push({
                        product: product.product,
                        productConsumption: Math.round(productConsumption),
                        items: product.items.map(item => ({
                            item: item.item,
                            consumption: Math.round(item.qty * productConsumption),
                            qty: item.qty,
                            price: item.price
                        }))
                    });
                });
                dailyData.push(saleDetails);
            });
            let monthlyProductsConsumption = Object.values(monthlyProductsConsumptionMap).map(product => {
                return {
                    product: product.product,
                    totalConsumption: Math.round(product.totalConsumption),
                    items: product.items.map(item => ({
                        item: item.item,
                        consumption: Math.round(item.consumption),
                        qty: item.qty,
                        price: item.price
                    }))
                };
            });

            localStorage.setItem('dailyData', JSON.stringify(monthlyProductsConsumption));

            const dayWiseItemConsumption = dailyData.map(sale => {
                const consumptionMap = {};

                sale.products.forEach(product => {
                    product.items.forEach(item => {
                        const { item: itemName, consumption, price } = item;

                        if (!consumptionMap[itemName]) {
                            consumptionMap[itemName] = { consumption: 0, price };
                        }

                        consumptionMap[itemName].consumption += consumption;
                    });
                });

                const itemsList = Object.keys(consumptionMap).map(itemName => ({
                    item: itemName,
                    consumption: consumptionMap[itemName].consumption,
                    price: consumptionMap[itemName].price
                }));

                return {
                    day: sale.day,
                    items: itemsList
                };
            });

            this.$emit('processed_consumption', dayWiseItemConsumption);

        }


    },

}

</script>