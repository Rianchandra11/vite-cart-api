<script>

export default {
    data() {
        return {
            grandTotal: 0,
            totalBarang_checkOut: 0,
            isLoading: true,
            cartEmpty: false
        }
    },
    methods: {
        async getGrandTotal(addDelay = false) {
            this.isLoading = true
            try {
                if (addDelay) {
                    await new Promise(resolve => setTimeout(resolve,1000))
                }
                const result = await( await fetch('http://localhost:3000/cart/grandtotal') ).json()
                this.grandTotal = result.grandtotal
                this.totalBarang_checkOut = result.total_data
                this.isLoading = false
            } catch(err) {
                console.log(err)
            }
        }
    },
    mounted() {
        this.getGrandTotal()
    }
}

</script>

<template>

    <div class="grand-total">
        <section style="width: 100%; padding: 0px 24px; display: flex; flex-direction: column; gap: 16px;">
            <span>
                <b style="font-weight: 800;">Ringkasan Belanja</b>
            </span>
            <div style="display: flex; justify-content: space-between; gap: 4px; align-items: center;">
                <span style="font-weight: 400; font-size: 0.875rem;" v-if="!isLoading">Total</span>
                <div class="shimmer" style="width: 100px; height: 1.3rem; border-radius: 1rem;" v-else></div>
                <span style="font-weight: 400; font-size: 1rem;" v-if="!isLoading" ref="textTotal">
                    <b :style="{fontWeight: grandTotal > 0 ? '800' : '400'}">{{ grandTotal > 0 ? 'Rp' + grandTotal.toLocaleString('id-ID') : '-' }}</b>
                </span>
                <div class="shimmer" style="width: 60px; height: 1.3rem; border-radius: 1rem;" v-else></div>
            </div>
            <hr class="border-line">
            <div style="padding: 8px 12px; display: flex; align-items: center; justify-content: space-between; gap: 8px; border-radius: 8px; border: 1px solid rgb(57,229,209); background-color: rgb(237,255,249); cursor: pointer;" v-if="!isLoading">
                <div style="display: flex; gap: 8px; align-items: center;" >
                    <img src="https://assets.tokopedia.net/asts/purchase-platform/icons/promo-coupon.svg" alt="ikon promo" style="width: 32px; height: 32px;">
                    <div>
                        <p style="color: rgb(33,33,33); font-size: 14px; margin: 0;">
                            <span>
                                <b style="font-weight: 800;">Pilih barang dulu sebelum pakai promo</b>
                            </span>
                        </p>
                    </div>
                </div>
                <div>
                    <svg class="nest-icon " width="24" height="24" fill="rgb(141,150,170)" viewBox="0 0 24 24">
                        <path d="M9.5 17.75a.75.75 0 0 1-.5-1.28L13.44 12 9 7.53a.75.75 0 0 1 1-1.06l5 5a.75.75 0 0 1 0 1.06l-5 5a.74.74 0 0 1-.5.22Z"></path>
                    </svg>
                </div>
            </div>
            <div v-else class="shimmer" style="width: 100%; height: 54px; border-radius: 8px;">

            </div>
        </section>
        <hr class="border-line">
        <div class="cartCheckOutBtn" style="width: 100%; padding: 0px 24px;">
            <button class="checkOutButton" :disabled="cartEmpty">
                <span style="margin: -1px; flex-grow: 1; padding: 0 8px; height: 40px;">
                    <span style="display: flex; justify-content: center; align-items: center; height: 100%; font-weight: bold;" v-if="!isLoading">
                        Beli {{totalBarang_checkOut ? '(' + totalBarang_checkOut + ')' : ''}}
                    </span>
                    <div style="display: flex; justify-content: center; height: 100%; align-items: center;" v-else>
                        <div class="spinner-border" role="status"  style="">
                            <span class="sr-only"></span>
                        </div>
                    </div>
                </span>
            </button>
        </div>
    </div>

</template>

<style>

.grand-total {
    background-color: white;
    border-radius: 12px;
    width: 384px;
    min-width: 200px;
    height: fit-content;
    margin-top: 60px;
    padding: 24px 0;
    display: flex;
    flex-direction: column;
    gap: 16px;
    position: sticky;
    top: 96px;
}

.spinner-border {
    --bs-spinner-width: 1.2rem;
    --bs-spinner-height: 1.2rem;
}

.checkOutButton {
    display: flex; 
    width: 100%; 
    cursor: pointer; 
    vertical-align: middle; 
    padding: 0; 
    border: 1px solid rgb(0,170,91); 
    background-color: rgb(0,170,91); 
    color: white; 
    border-radius: 12px; 
    font-size: 0.875rem;
}

.checkOutButton:disabled {
    cursor: not-allowed;
    border-color: rgb(240,243,247);
    background: rgb(240,243,247);
    color: rgb(170,180,200);
}

</style>