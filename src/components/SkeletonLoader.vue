<script>
import CheckBox from './CheckBox.vue';


export default {
    components: {
        CheckBox
    },
    props: {
        isLoading: Boolean
    },
    methods: {
        getRandomNumber() {
            return Math.floor(Math.random()*3)+1
        },
        getRandomWidthTitle() {
            return (Math.random()*100).toFixed(2)
        },
        getRandomWidthSubtitle() {
            return (Math.random()*40).toFixed(2)
        }
    }
}

</script>

<template>

    <div>

        <div style="padding-top: 9px;" v-for="iterate in 5">
            <div class="shop-header">
                <div class="margin-checkbox" data-margin-store="true">
                    <CheckBox />
                </div>
                <div style="width: 100%; margin: 16px 24px 0px 0px;">
                    <div style="width: 120px; height: 17px; border-radius: 20px;" class="shimmer"></div>
                </div>
            </div>
            <div style="width: 100%;" v-for="iterate in getRandomNumber()">
                <div style="display: flex; gap: 16px;">
                    <div style="margin: 16px 0 16px 24px">
                        <CheckBox />
                    </div>
                    <div style="width: 100%;">
                        <div style="margin: 16px 24px 16px 0px;">
                            <div style="display: flex; gap: 16px;">
                                <div class="discount-label">
                                    <div class="product-img shimmer"></div>
                                </div>
                                <div style="display: flex; flex-direction: column; width: 100%; justify-content: space-between;">
                                    <div style="display: flex; gap: 16px;">
                                        <section style="display: flex; gap: 8px; flex-direction: column; flex: 1 1 0%;">
                                            <div class="shimmer" :style="{width: `calc(300px + ${getRandomWidthTitle()}px)`, height: '1.1rem', borderRadius: '50px'}"></div>
                                            <div class="shimmer" :style="{width: `calc(80px + ${getRandomWidthSubtitle()}px)`, height: '0.875rem', borderRadius: '1rem'}"></div>
                                        </section>
                                        <section style="display: flex; flex-direction: column; gap: 4px; align-items: flex-end;">
                                            <div style="display: flex; gap: 4px; align-items: center;">
                                                <div class="shimmer" :style="{width: `calc(110px + ${getRandomWidthSubtitle()}px)`, height: '1.1rem', borderRadius: '1rem'}"></div>
                                            </div>
                                        </section>
                                    </div>

                                    <div style="display: flex; gap: 16px; align-self: end;">
                                        <div style="display: flex; flex-direction: column; box-sizing: border-box; align-items: flex-end; gap: 2px;">
                                            <div class="shimmer" style="width: 95px; height: 32px; border-radius: 8px;"></div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        
                        <hr class="border-line">
                    </div>
                </div>
            </div>
            <hr class="border-line" data-border-lg v-if="iterate != 5">
        </div>

    </div>

</template>

<style scoped>

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

.product-img {
    margin: 0;
    width: 80px;
    height: 80px;
    border-radius: 8px;
}

</style>