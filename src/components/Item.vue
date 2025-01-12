<script>
import CheckBox from './CheckBox.vue';

export default {
    props: {
        product: Object,
        index: Number,
        totalProductShop: Number,
    },
    data() {
        return{
            checked: false,
            quantity: 0,
            quantityFocus: false,
            quantityAllow: true,
            grandTotalComponent: this.$parent.$parent.$parent.$refs.grandTotalComponent,
            processDelay: null,
            catatan: null
        }
    },
    components: {
        CheckBox
    },
    created() {
        this.checked = Boolean(this.product.Selected)
        this.quantity = this.product.Quantity
        this.catatan = this.product.Catatan
    },
    methods: {
        async updateQty(value, id) {
            if (this.quantityAllow) {
                try {
                    if (this.quantity == 0) return
                    if (this.processDelay) clearTimeout(this.processDelay)
                    const response = await fetch(`http://localhost:3000/cart/quantity/${id}`, {
                        method: 'PUT',
                        headers: {
                            'Content-Type': 'application/json'
                        },
                        body: JSON.stringify({
                            "operator": typeof(value) == 'string' ? value : null,
                            "value": typeof(value) == 'number'  ? value : null
                        })
                    })
                    if (!response.ok) {
                        throw await response.json()
                    }
                    const result = await response.json()
                    const currentQuantity = result.updatedQuantity
                    this.processDelay = setTimeout(() => {
                        this.quantityAllow = false
                        this.grandTotalComponent.getGrandTotal(true)
                        .then( () => {
                            this.quantity = currentQuantity
                            this.quantityFocus = false
                            this.quantityAllow = true
                        })
                        
                    }, 1000)
                } catch(err) {
                    console.log(err)
                }
            }
        },

        async clickState(event) {
            try {
                const response = await fetch(`http://localhost:3000/cart/${event}/${this.product.ID}`, {
                    method: "PUT"
                })
                if (!response.ok) {
                    throw await response.json()
                }
                if (event == "wishlist") {
                    this.product.Wishlist = !this.product.Wishlist
                } else if (event == "selected") {
                    await this.$parent.$parent.updateGrandtotal()
                }
            } catch(err) {
                console.log(err)
            }
        },

        quantityProhibitedValue() {
            if (this.quantity > this.product.stok) {
                this.quantity = this.product.stok
            } else if (this.quantity < 1) {
                this.quantity = 1
            }
            this.updateQty(this.quantity, this.product.ID)
        },

        addNote() {
            const prev_note = this.catatan
            this.$parent.$parent.$parent.showPopUpNote(!prev_note ? "" : prev_note, (ref) => {

                const {note, simpan} = {...ref}

                simpan.addEventListener('click', async () => {
                    const new_note = note.value
                    try {
                        const response  = await fetch(`http://localhost:3000/cart/note/${this.product.ID}`, {
                            method: "PUT",
                            headers: {
                                'Content-Type': 'application/json'
                            },
                            body: JSON.stringify({
                                note: new_note == "" ? null : new_note
                            })
                        })
                        if (!response.ok) {
                            throw await response.json()
                        }
                        this.catatan = new_note
                        console.log(new_note)
                    } catch(err) {
                        console.log(err)
                    }
                })
            })
        },

        async deleteItem() {
            try {
                const response = await fetch(`http://localhost:3000/cart/delete/${this.product.ID}`, {
                    method: "DELETE"
                })
                if (!response.ok) {
                    throw await response.json()
                }
                await this.$parent.$parent.getCartItems(true)
            } catch(err) {
                console.log(err)
            }
        }
        
    }, watch: {
        product() {
            this.checked = Boolean(this.product.Selected)
        }
    }
}

</script>

<template>

    <div style="width: 100%;">
        <div style="display: flex; gap: 16px;">
            <div style="margin: 16px 0 16px 24px">
                <CheckBox :selected="checked" @changeState="(state) => {checked = state ; clickState('selected'); $emit('changeState')}"/>
            </div>
            <div style="width: 100%;">
                <div style="margin: 16px 24px 16px 0px;">
                    <div style="display: flex; gap: 16px;">
                        <div class="discount-label">
                            <img :src="`http://localhost:3000/image/Gambar_Barang/${product.ID}`" alt="" class="product-img">
                            <span v-if="product.Diskon > 0">
                                {{ product.Diskon }}%
                            </span>
                        </div>
                        <div style="display: flex; flex-direction: column; width: 100%; justify-content: space-between; gap: 4px;">
                            <div style="display: flex; gap: 16px;">
                                <section style="display: flex; gap: 8px; flex-direction: column; flex: 1 1 0%;">
                                    <p class="remain-stock" v-if="product.stok <= 5"><b style="font-weight: 800;">Sisa {{ product.stok }}</b></p>
                                    <span class="product-title">{{ product.Nama_Barang }}</span>
                                    <span class="product-variant" v-if="product.variant">{{ product.variant }}</span>
                                </section>
                                <section style="display: flex; flex-direction: column; gap: 4px; align-items: flex-end;">
                                    <div style="display: flex; gap: 4px; align-items: center;">
                                        <!-- <span class="voucher"></span> -->
                                        <span data-voucher="false" class="total-price">Rp{{ (Math.ceil(product.Harga - product.Harga*product.Diskon/100)).toLocaleString("id-ID") }}</span>
                                    </div>
                                    <span class="discount" v-if="product.Diskon > 0">
                                        <s>Rp{{ product.Harga.toLocaleString("id-Id") }}</s>
                                    </span>
                                </section>
                            </div>

                            <div style="display: flex; gap: 16px; align-self: end;">

                                <div v-if="catatan" @click="addNote()" style="max-width: 435px; border: 1px solid rgb(191,201,217); height: 32px; display: flex; align-items: center; padding: 0 6px 0 10px; border-radius: 50px; gap: 6px; cursor: pointer;">
                                    <span style="font-size: 0.875rem; color: rgb(141,150,170); white-space: nowrap;overflow: hidden; text-overflow: ellipsis;">{{ catatan }}</span>
                                    <button class="button-icon">
                                        <svg class="nest-icon " width="24" height="24" fill="rgb(141,150,170)" viewBox="0 0 24 24">
                                            <path fill-rule="evenodd" clip-rule="evenodd" d="M11.12 19.23H6.59c-1 0-1.8-.81-1.8-1.8V5.56c0-1 .81-1.8 1.8-1.8h9.14c.99 0 1.8.81 1.8 1.8v4.7c0 .41.34.75.75.75s.75-.34.75-.75v-4.7c0-1.82-1.48-3.3-3.3-3.3H6.59c-1.82 0-3.3 1.48-3.3 3.3v11.86c0 1.82 1.48 3.3 3.3 3.3h4.53c.41 0 .75-.34.75-.75s-.34-.75-.75-.75v.01ZM15.75 5.9h-9.6c-.41 0-.75.34-.75.75s.34.75.75.75h9.6c.41 0 .75-.34.75-.75s-.34-.75-.75-.75Zm-9.6 3.33h9.6c.41 0 .75.34.75.75s-.34.75-.75.75h-9.6c-.41 0-.75-.34-.75-.75s.34-.75.75-.75Zm6.5 3.32h-6.5c-.41 0-.75.34-.75.75s.34.75.75.75h6.5c.41 0 .75-.34.75-.75s-.34-.75-.75-.75Zm7.7-.98 1.4 1.05v.01c.26.2.43.48.48.81.05.32-.03.65-.23.91l-4.88 6.53c-.16.22-.4.38-.67.45l-1.65.43a.977.977 0 0 1-.84-.17.957.957 0 0 1-.39-.76l-.06-1.7c-.01-.28.07-.56.24-.78l4.88-6.53a1.23 1.23 0 0 1 1.72-.25Zm-5.32 8.57.94-.24v-.01l2.64-3.53-.97-.72L15 19.17l.03.97Zm3.51-5.69.97.72 1.13-1.51-.97-.72-1.13 1.51Z"></path>
                                        </svg>
                                    </button>
                                </div>

                                <button v-else class="button-icon" @click="addNote()">
                                    <svg class="nest-icon " width="24" height="24" fill="rgb(141,150,170)" viewBox="0 0 24 24">
                                        <path fill-rule="evenodd" clip-rule="evenodd" d="M11.12 19.23H6.59c-1 0-1.8-.81-1.8-1.8V5.56c0-1 .81-1.8 1.8-1.8h9.14c.99 0 1.8.81 1.8 1.8v4.7c0 .41.34.75.75.75s.75-.34.75-.75v-4.7c0-1.82-1.48-3.3-3.3-3.3H6.59c-1.82 0-3.3 1.48-3.3 3.3v11.86c0 1.82 1.48 3.3 3.3 3.3h4.53c.41 0 .75-.34.75-.75s-.34-.75-.75-.75v.01ZM15.75 5.9h-9.6c-.41 0-.75.34-.75.75s.34.75.75.75h9.6c.41 0 .75-.34.75-.75s-.34-.75-.75-.75Zm-9.6 3.33h9.6c.41 0 .75.34.75.75s-.34.75-.75.75h-9.6c-.41 0-.75-.34-.75-.75s.34-.75.75-.75Zm6.5 3.32h-6.5c-.41 0-.75.34-.75.75s.34.75.75.75h6.5c.41 0 .75-.34.75-.75s-.34-.75-.75-.75Zm7.7-.98 1.4 1.05v.01c.26.2.43.48.48.81.05.32-.03.65-.23.91l-4.88 6.53c-.16.22-.4.38-.67.45l-1.65.43a.977.977 0 0 1-.84-.17.957.957 0 0 1-.39-.76l-.06-1.7c-.01-.28.07-.56.24-.78l4.88-6.53a1.23 1.23 0 0 1 1.72-.25Zm-5.32 8.57.94-.24v-.01l2.64-3.53-.97-.72L15 19.17l.03.97Zm3.51-5.69.97.72 1.13-1.51-.97-.72-1.13 1.51Z"></path>
                                    </svg>
                                </button>

                                <button class="button-icon" @click="clickState('wishlist')">
                                    <svg class="nest-icon " width="24" height="24" :fill="product.Wishlist ? 'rgb(249,77,99)' : 'rgb(141,150,170)'"  viewBox="0 0 24 24" data-testid="cartAddWishlist">
                                        <path v-if="!product.Wishlist" d="M12.11 20.81a1.61 1.61 0 0 1-.92-.28c-2.14-1.28-6-4-7.92-7.64a6.8 6.8 0 0 1 0-7.12 5.39 5.39 0 0 1 4.6-2.54A5.1 5.1 0 0 1 12 5.55a5.14 5.14 0 0 1 4.24-2.32 5.5 5.5 0 0 1 4.56 2.56 7.62 7.62 0 0 1 .15 7c-2.31 4.17-7 7.15-8 7.7a1.63 1.63 0 0 1-.84.32ZM7.87 4.73a3.89 3.89 0 0 0-3.4 1.87c-.18.27-1.6 2.45.13 5.59 1.7 3.32 5.4 5.86 7.4 7.08a.19.19 0 0 0 .2 0c.56-.34 5.29-3.25 7.43-7.1a6.11 6.11 0 0 0-.09-5.6 4 4 0 0 0-3.29-1.86 4.12 4.12 0 0 0-3.57 2.61L12 8.68l-.67-1.34c-.84-1.68-2.07-2.61-3.46-2.61Z"></path>
                                        <path v-else d="M20.81 5.81a5.5 5.5 0 0 0-4.56-2.56A5.14 5.14 0 0 0 12 5.55a5.1 5.1 0 0 0-4.13-2.32 5.39 5.39 0 0 0-4.65 2.54 6.8 6.8 0 0 0 0 7.12c1.87 3.64 5.78 6.36 7.92 7.64.271.185.592.283.92.28a1.63 1.63 0 0 0 .89-.26c.92-.55 5.65-3.53 8-7.7a7.62 7.62 0 0 0-.14-7.04Z"></path>
                                    </svg>
                                </button>
                                
                                <button class="button-icon" style="display: flex; align-items: center;" @click="deleteItem()">
                                    <svg class="nest-icon " width="24" height="24" fill="rgb(141,150,170)" viewBox="0 0 24 24" data-testid="cartBtnDelete">
                                        <path fill-rule="evenodd" clip-rule="evenodd" d="M20 5.25h-5.24a.7.7 0 0 0 .05-.25 2.76 2.76 0 0 0-5.52 0 .7.7 0 0 0 .05.25H4a.75.75 0 0 0 0 1.5h1.25V20A1.76 1.76 0 0 0 7 21.75h10A1.76 1.76 0 0 0 18.75 20V6.75H20a.75.75 0 1 0 0-1.5ZM10.79 5a1.26 1.26 0 1 1 2.52 0 .996.996 0 0 0 0 .25h-2.57a.7.7 0 0 0 .05-.25Zm6.46 15a.25.25 0 0 1-.25.25H7a.25.25 0 0 1-.25-.25V6.75h10.5V20ZM10 17.74a.75.75 0 0 0 .75-.75V10.1a.75.75 0 1 0-1.5 0V17a.74.74 0 0 0 .75.74Zm4.349-.054a.74.74 0 0 1-.289.054.75.75 0 0 1-.75-.74v-6.9a.75.75 0 1 1 1.5 0v6.89a.741.741 0 0 1-.461.696Z"></path>
                                    </svg>
                                </button>
                                <div style="display: flex; flex-direction: column; box-sizing: border-box; align-items: flex-end; gap: 2px;">
                                    <div class="buttonQtyWrapper" :style="quantityFocus ? {borderColor: '#00AA5B'} : ''">
                                        <div style="display: flex; margin: -1px; align-items: center; padding: 0;">
                                            <div style="display: flex; gap: 2px;">
                                                <button class="handler" @click="() => {quantity -= 1 ; quantityFocus = true ; updateQty('kurang', product.ID)}" :style="quantityFocus ? {color: '#00AA5B'} : ''" :disabled="quantity == 1 ? true : false">
                                                    <svg width="100%" height="100%" viewBox="0 0 24 24" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
                                                        <path d="M17.41 12.5H6.75c-.41 0-.75-.34-.75-.75s.34-.75.75-.75h10.66c.41 0 .75.34.75.75s-.34.75-.75.75z"></path>
                                                    </svg>
                                                </button>
                                                <input type="number" min="1" :max="product.stok" v-model="quantity" class="quantity" @focusout="quantityProhibitedValue()" @input="() => {quantity < 1 ? quantity = 0 : null ; updateQty(quantity, product.ID)}">
                                            </div>
                                            <button class="handler" :disabled="quantity == product.stok ? true : false" @click="() => {quantity += 1 ; quantityFocus = true; updateQty('tambah', product.ID)}" :style="quantityFocus ? {color: '#00AA5B'} : ''">
                                                <svg width="100%" height="100%" viewBox="0 0 24 24" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
                                                    <path d="M12.75 6.67a.75.75 0 1 0-1.5 0v4.58H6.67a.75.75 0 1 0 0 1.5h4.58v4.58a.75.75 0 1 0 1.5 0v-4.58h4.58a.75.75 0 1 0 0-1.5h-4.58V6.67z"></path>
                                                </svg>
                                            </button>
                                        </div>
                                    </div>
                                    <span style="color: rgb(109,117,136); font-weight: 400; font-size: 0.75rem; line-height: 1.33333; width: fit-content;" v-if="quantity > product.stok || quantity < 1"> {{ quantity > product.stok ? `Maks. Beli ${product.stok}` : 'Min. Beli 1'}}</span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <hr v-if="index != totalProductShop" class="border-line">
            </div>
        </div>
    </div>

</template>

<style>

.product-img {
    margin: 0;
    width: 80px;
    height: 80px;
    border: 1px solid rgb(240,243,247);
    border-radius: 8px;
    object-fit: cover;
}

.remain-stock {
    color: rgb(255,127,23);
    font-weight: 400;
    font-size: 0.625rem;
    line-height: 1.2;
    margin: 0;
    padding: 0;
}

.product-title {
    font-weight: 400;
    font-size: 0.875rem;
    line-height: 1.57143;
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: pre-wrap;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    line-clamp: 2;
    -webkit-box-orient: vertical;
    word-break: break-word;
    color: rgb(33,33,33);
}

.product-variant {
    color: rgb(109,117,136);
    font-size: 0.875rem;
    line-height: 1.28571;
    font-weight: 400;
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: pre-wrap;
    display: -webkit-box;
    -webkit-line-clamp: 1;
    line-clamp: 1;
    -webkit-box-orient: vertical;
    word-break: break-word;
}

.total-price {
    color: rgb(33,33,33);
    font-size: 1rem;
    font-weight: bold;
    line-height: 1.25;
}

.total-price[data-voucher="true"] {
    color: rgb(254, 44, 85);
}

.voucher {
    width: 20px;
    height: 20px;
    background-size: contain;
    background-image: url("https://images.tokopedia.net/img/ic_promo-discount.png");
}

.discount {
    color: rgb(109,117,136);
    font-weight: 400;
    font-size: 0.875rem;
    line-height: 1.28571;
}

.discount-label {
    position: relative;
    display: block;
    --ribbon-fold-size: 4px;
}

.discount-label span {
    border-radius: 8px 12px 12px 0;
    left: calc(0px - var(--ribbon-fold-size));
    top: 8px;
    padding: 4px 8px;
    font-size: 0.625rem;
    line-height: 1.2;
    display: block;
    position: absolute;
    background-color: rgb(249, 77, 99);
    color: white;
    font-weight: 800;
    box-sizing: border-box;
}

.discount-label span::before {
    left: 0px;
    border-bottom-left-radius: 100%;
    content: "";
    display: block;
    position: absolute;
    top: 100%;
    width: var(--ribbon-fold-size);
    height: var(--ribbon-fold-size);
    background-color: rgb(179, 29, 64);
}

.button-icon {
    background: none;
    cursor: pointer;
    border: none;
    margin: 0;
    padding: 0;
    height: 32px;
}

.handler {
    cursor: pointer;
    background: none;
    border: none;
    width: 32px;
    height: 32px;
    padding: 4px;
}

.handler:disabled {
    color: rgb(191,201,217);
}

.quantity {
    color: rgb(33,33,33);
    width: 3ch;
    background: none;
    border: none;
    padding: 0;
    text-align: center;
    outline: none;
    font-family: inherit;
    line-height: 1.57143;
}

.quantity::-webkit-outer-spin-button,
.quantity::-webkit-inner-spin-button {
    -webkit-appearance: none;
    margin: 0;
}

.buttonQtyWrapper {
    border-radius: 16px; 
    border: 1px solid rgb(191,201,217); 
    font-size: 0.75rem;
}

.buttonQtyWrapper:has(.quantity:focus) {
    border-color: #00AA5B;
}

.buttonQtyWrapper:has(.quantity:focus) .handler {
    color: #00AA5B;
}

</style>