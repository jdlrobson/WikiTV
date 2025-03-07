<template>
    <div :class="className">
        <input type="range" v-model="num"
            min="0" max="9" step="1">
    </div>
</template>
<script>
import { ref, defineComponent } from 'vue';
export default defineComponent( {
    props: {
        defaultValue: {
            type: Number
        }
    },
    computed: {
        className() {
            return {
                dial: true,
                [`dial-${this.num.value}`]: true
            };
        }
    },
    setup( props ) {
        const { defaultValue } = props;
        const num = ref( defaultValue );
        return {
            num
        }
    }
} );
</script>
<style>
.dial {
    background: white;;
    border: solid 1px black;
    border-radius: 20px;
    width: 20px;
    height: 20px;
    position: relative;
}
.dial input {
    visibility: hidden;
    width: 20px;
    height: 20px;
    position: absolute;
    left: 0;
    right: 0;
    bottom: 0;
    top: 0;
    margin-left: 0px;
    margin-top: 0;
}
.dial:before {
    content: '.';
}
.dial-0 {
    transform: rotate(0deg);
}
.dial-1 {
    transform: rotate(40deg);
}
.dial-2 {
    transform: rotate(80deg);
}
.dial-3 {
    transform: rotate(120deg);
}
.dial-4 {
    transform: rotate(160deg);
}
.dial-5 {
    transform: rotate(200deg);
}
.dial-6 {
    transform: rotate(240deg);
}
.dial-7 {
    transform: rotate(280deg);
}
.dial-8 {
    transform: rotate(320deg);
}
.dial-9 {
    transform: rotate(360deg);
}
</style>