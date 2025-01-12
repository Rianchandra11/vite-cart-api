<script>
import CheckBox from './CheckBox.vue'
import Item from './Item.vue';

export default {
    data() {
        return{
            selectShop: false
        }
    },
    components: {
        CheckBox,
        Item
    },
    props: {
        itemToko: Array,
        namaToko: String,
        index: Number,
        totalShop: Number
    },
    methods: {
        checkShopItem() {
            let sum = 0
            this.$refs.items.forEach(e => {
                if (e.checked) sum += 1
            })
            sum == this.itemToko.length ? this.selectShop = true : this.selectShop = false
            return sum
        },
        async selectAllShopItem(state) {
            try {
                const response = await fetch(`http://localhost:3000/cart/selected/shop/${this.itemToko[0].id_shop}/${state}`, {
                    method: "PUT"
                })
                if (!response.ok) {
                    throw await response.json()
                }
                this.selectShop = state
                this.$refs.items.forEach(e => {
                    e.checked = this.selectShop
                })
                await this.$parent.updateGrandtotal()
            } catch (err) {
                console.log(err)
            }
        }
    },
    mounted() {
        this.checkShopItem()
    },
    watch: {
        selectShop() {
            this.$emit("changeState")
            this.$emit("totalSelected")
        }
    }
}

</script>

<template>

    <div>

        <div style="padding-top: 9px;">
            <div class="shop-header">
                <div class="margin-checkbox" data-margin-store="true">
                    <CheckBox :selected="selectShop" @changeState="(state) => selectAllShopItem(state)"/>
                </div>
                <div style="width: 100%; display: flex; align-items: end;">
                    <div style="margin-right: 24px;">
                        <div>
                            <section style="display: flex; gap: 9px; align-items: center;">
                                <img :src="`http://localhost:3000/image/Logo_Toko/${itemToko[0].ID}`" alt="shop-badge" class="shop-badge">
                                <span style="font-weight: bold; font-size: 0.875rem;">{{ namaToko }}</span>
                            </section>
                        </div>
                    </div>
                </div>
            </div>
            <Item v-for="(item, index) in itemToko" :key="item.ID" :product="item" :index="index" :totalProductShop="itemToko.length-1" ref="items" @changeState="() => {checkShopItem() ; $emit('totalSelected')}"/>
            <hr class="border-line" data-border-lg v-if="index != totalShop">
        </div>

    </div>

</template>

<style>

.shop-header {
    display: flex;
    gap: 16px;
}

.margin-checkbox {
    margin: 16px 0 0 24px;
}

.margin-checkbox[data-margin-store="false"] {
    align-self: flex-start
}

.shop-badge{
    width: 30px;
    height: 30px;
}

</style>
