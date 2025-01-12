<script>

import CheckBox from './CheckBox.vue'
import ListItem from './Shop.vue'
import SkeletonLoader from './SkeletonLoader.vue';

export default {
    data() {
        return {
            listCart: [],
            totalItem: 0,
            selectAll: false,
            totalSelectedItem: 0,
            isLoading: true
        }
    },
    components: {
        CheckBox,
        ListItem,
        SkeletonLoader
    },
    methods: {
        async getCartItems(refreshGrandTotal = false) {
            try {
                const response = await (await fetch("http://localhost:3000/cart")).json()
                this.totalItem = response.total_item
                this.listCart = response.available_group
                this.isLoading = false
                if (refreshGrandTotal) {
                    await this.updateGrandtotal()
                }
            } catch(error) {
                console.log(error)
            }
        },

        checkSelectShop() {
            const totalSelectedShop = this.$refs.shops.reduce((sum, shop) => {
                if (shop.selectShop) return sum + 1
                else return sum
            },0)
            totalSelectedShop == Object.keys(this.listCart).length ? this.selectAll = true : this.selectAll = false
        },

        async selectAllShop() {
            this.selectAll = !this.selectAll
            try {
                const response = await fetch(`http://localhost:3000/cart/selected/all/${this.selectAll}`, {
                    method: "PUT"
                })
                if (!response.ok) {
                    throw await response.json()
                }
                await this.getCartItems()
                await this.updateGrandtotal()
            } catch(err) {
                console.log(err)
            }
        },

        checkSelectedItem() {
            this.totalSelectedItem = this.$refs.shops.reduce((total, shop) => {
                return total += shop.checkShopItem()
            }, 0)
        },

        async updateGrandtotal() {
            try {
                const response = await this.$parent.$refs.grandTotalComponent.getGrandTotal(true)
            } catch(err) {
                console.log(err)
            }
        },

        async deleteSelected() {
            try {
                const response = await fetch("http://localhost:3000/cart/delete/selected", {
                    method: "DELETE"
                })
                if (!response.ok) {
                    throw await response.json()
                }
                await this.getCartItems(true)
            } catch(err) {
                console.log(err)
            }
        }
    },  
    created() {
        this.getCartItems()
    },
    updated() {
        if (this.$refs.shops) {
            this.checkSelectShop()
            this.checkSelectedItem()
        }
    },
    watch: {
        listCart() {
            if(Object.keys(this.listCart).length == 0) {
                this.$parent.$refs.grandTotalComponent.cartEmpty = true
            } else {
                this.$parent.$refs.grandTotalComponent.cartEmpty = false
            }
        }
    }
}

</script>


<template>

    <div class="list-cart" v-if="isLoading">

        <div class="select-all">
            <div style="margin: 24px 0px 16px 24px;">
                <CheckBox/>
            </div>
            <div style="width: 100%;">
                <div style="margin: 24px 24px 16px 0;">
                    <div style="display: flex; align-items: center;">
                        <div style="width: 120px; height: 17px; border-radius: 20px;" class="shimmer"></div>
                    </div>
                </div>
            </div> 
        </div>
        <hr class="border-line" data-border-lg>
        <SkeletonLoader/>

    </div>

    <div class="list-cart" v-else>

        <div v-if="Object.keys(listCart).length > 0">
            <div class="select-all">
                <div style="margin: 24px 0px 16px 24px;">
                    <CheckBox :selected="selectAll" @changeState="selectAllShop()"/>
                </div>
                <div style="width: 100%;">
                    <div style="margin: 24px 24px 16px 0;">
                        <div style="display: flex; justify-content: space-between;">
                            <div style="display: flex; gap: 4px">
                                <span class="title" style="color: rgb(33,33,33);">Pilih Semua</span>
                                <span class="title" style="font-weight: normal;">({{ totalItem }})</span>
                            </div>
                            <button style="background: none; border: none;" v-if="totalSelectedItem > 0" @click="deleteSelected()">
                                <span style="color: rgb(0,170,91); font-size: 0.875rem;"><b>Hapus</b></span>
                            </button>
                        </div>
                    </div>
                </div> 
            </div>
            <hr class="border-line" data-border-lg>
            <ListItem v-for="(value, key, index) in listCart" :key="value[0].id_shop" :itemToko="value" :namaToko="key" :index="index" :totalShop="Object.keys(listCart).length-1" ref="shops" @changeState="checkSelectShop()" @totalSelected="checkSelectedItem()"/>
        </div>

        <div v-else>
            <div style="display: flex; justify-content: center; align-items: center; padding: 32px 0; ">
                <div style="display: flex; gap: 16px;">
                    <img src="https://assets.tokopedia.net/assets-tokopedia-lite/v2/backfunnel/kratos/4d27af6a.svg">
                    <div style="display: flex; flex-direction: column; gap: 4px; justify-content: center;">
                        <span style="font-size: 1.25rem; font-weight: 800;">Maaf, keranjang belanja anda kosong</span>
                        <span style="color: rgb(109,117,136); font-size: 0.875rem;">Tekan tombol reset diatas</span>
                    </div>
                </div>
            </div>
        </div>

    </div>

</template>

<style>

.list-cart {
    width: 100%;
    background-color: white;
    border-radius: 12px;
    
}

.select-all {
    display: flex;
    gap: 16px;
}

</style>