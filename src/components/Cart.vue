<script>

import ListKeranjang from './ListKeranjang.vue';
import Grandtotal from './Grandtotal.vue';
import Heading from './Heading.vue';

export default {
    components: {
        ListKeranjang,
        Grandtotal,
        Heading
    },
    data() {
        return {
            catatan: "",
            popUpNote: false,
            isLoading: false
        }
    },
    methods: {
        async showPopUpNote(prev_note, callback) {
            this.popUpNote = true
            this.catatan = prev_note
            await this.$nextTick()
            return callback({"note": this.$refs.noteText, "simpan": this.$refs.simpan})
        },

        async resetCart() {
            try {
                this.isLoading = true
                const response = await fetch("http://localhost:3000/cart/reset", {
                    method: "POST"
                })
                if (!response.ok) {
                    throw await response.json()
                }
                this.$refs.listKeranjang.isLoading = true
                await this.$refs.listKeranjang.getCartItems(true)
                this.isLoading = false
            } catch(err) {
                console.log(err)
            }
        }
    }
}

</script>


<template>

<Heading @click="popUpNote=false"/>

<div>
    <div style="display: grid; place-items: center;">
        <div id="content">
            <div class="cart">
                <div style="display: flex; justify-content: space-between; align-items: center;">
                    <span style="font-weight: 800; font-size: 1.5rem">Keranjang</span>
                    <button type="button" class="btn btn-outline-danger" style="width: 88px; height: 35px;" @click="resetCart()" :disabled="isLoading">
                        <div v-if="!isLoading" style="display: flex; gap: 4px; align-items: center;">
                            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-arrow-clockwise" viewBox="0 0 16 16">
                                <path fill-rule="evenodd" d="M8 3a5 5 0 1 0 4.546 2.914.5.5 0 0 1 .908-.417A6 6 0 1 1 8 2z"/>
                                <path d="M8 4.466V.534a.25.25 0 0 1 .41-.192l2.36 1.966c.12.1.12.284 0 .384L8.41 4.658A.25.25 0 0 1 8 4.466"/>
                            </svg>
                            <span style="font-size: 0.875rem; font-weight: 800;">Reset</span>
                        </div>
                        <div class="spinner-border" role="status"  style="" v-else>
                            <span class="sr-only"></span>
                        </div>
                    </button>
                </div>
                <ListKeranjang ref="listKeranjang"/>
            </div>

            <Grandtotal ref="grandTotalComponent"/>
        </div>
    </div>

    <div v-if="popUpNote" style="width: 100%; position: fixed; top: 0; bottom: 0; display: flex; z-index: 5; justify-content: center; align-items: center; background-color: rgb(0,0,0,.3);">
        <div style="width: 500px; background-color: white; border-radius: 16px; padding: 24px;">
            <span style="font-size: 1rem;">
                <b style="font-weight: 800;">Catatan Pesanan</b>
            </span>
            <hr>
            <div style="position: relative;">
                <textarea ref="noteText" name="" id="" v-model="catatan" maxlength="144" placeholder="Jangan Masukkan data pribadi, ya." @keydown="(event) => event.key === 'Enter' ? event.preventDefault() : null"></textarea>
                <span style="position: absolute; border-radius: 8px 0 4px 0px; right: 7px; bottom: 7px; font-size: 0.775rem; color: rgb(170,180,200); padding-left: 4px; padding-top: 4px; background-color: white;">{{ catatan.length }}/144</span>
            </div>
            <div style="display: flex; gap: 14px; width: 100%; justify-content: flex-end; margin-top: 10px;">
                <button class="noteButton" style="color: #03AC0E ;background-color: white; border: 2px solid #03AC0E;" @click="popUpNote=false">Batal</button>
                <button class="noteButton" style="color: white ;background-color: #03AC0E; border: none;" ref="simpan" @click="popUpNote=false">Simpan</button>
            </div>
        </div>
    </div>
</div>



</template>

<style>

.cart {
  width: 800px;
  display: flex;
  gap: 24px;
  flex-direction: column;
}

textarea {
    width: 100%;
    height: 200px;
    padding: 8px 8px 14px;
    font-size: 1rem;
    border-radius: 5px;
    resize: none;
    border-color: #c4c4c4;
}

textarea:focus {
    outline: none;
}

textarea::-webkit-scrollbar {
    display: none;
}

.noteButton {
    width: 90px; 
    height: 40px; 
    font-size: 1rem; 
    font-weight: 800;
    border-radius: 7px;
    border: 1px solid #c4c4c4;
}

</style>