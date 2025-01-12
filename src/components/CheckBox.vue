<script>

export default {
    props: {
        selected: Boolean
    },
    methods: {
        changeState() {
            this.$emit("changeState", !this.selected)
        }
    }
}

</script>

<template>
    <input class="form-check-input" type="checkbox" value="" id="flexCheckDefault" :checked="selected" @click="changeState()" :disabled="Boolean($parent.isLoading)">
</template>

<style>

.form-check-input {
    width: 20px;
    height: 20px;
    cursor: pointer;
    box-shadow: none;
    border-color: #8D96AA;
    margin: 0;
}

.form-check-input:focus {
    box-shadow: none;
    border: 1px solid #8D96AA;
}


.form-check-input:checked {
    background-color: #00AA5B;
    border: none;
}

.form-check-input:disabled {
    cursor: not-allowed;
    pointer-events: all !important;
}

</style>