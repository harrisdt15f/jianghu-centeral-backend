<template>
    <span :class="['v-input', size]">
        <input
            :class="[disabled?'disabled':'']"
            :type="type==='password'? 'password':'text'"
            ref="input"
            :placeholder="placeholder"
            v-model="val"
            @input="input"
            @keyup.enter="pressEnter"
            @keyup="keyup"
            @focus="focus"
            @blur="$emit('blur')"
            :maxlength="maxlength"
            :disabled="disabled"
            :autocomplete="autocomplete"
        />
        <i v-if="icon" :class="['iconfont', icon,'icon-input']"></i>

        <span v-show="showerr||(required&&!value)" class="error-message">
            <i class="iconfont iconjinggao1-"></i>
            {{errmsg}}
        </span>
    </span>
</template>

<script>
export default {
    name: 'Input',
    props: {
        disabled: Boolean,
        type: {
            type: String,
            default: 'text'
        },
        icon: String,
        size: {
            type: String,
            default: 'normal'
        },
        limit: {
            type: String,
            default: ''
        },
        placeholder: String,
        value: [String, Number],
        maxlength: [Number, String],
        autocomplete: {
            type: String
        },
        showerr: {
            type: Boolean,
            default: false
        },
        errmsg: {
            type: String,
            default: ''
        },
        // 当required为true时, 值为空,就会提示
        required: {
            type: Boolean,
            default: false
        }
    },
    model: {
        prop: 'value',
        event: 'keyup' // 原来是input
    },
    data() {
        return {
            val: '',
            regs: {
                number: /[^\-?\d.]/g, // 数字
                'p-number': /[^\d.]/g, // 正数
                integer: /[^\-?\d]/g, // 整数
                'p-integer': /\D/g, //  正整数
                'no-zh-cn': /[\u4E00-\u9FA5]*/g, // 非中文
                'en-num': /[\W_]/g, // 字母,数字
                en: /[\W_0-9]/g, // 字母
                word: /[^\w\-]/g // 字母数字,下划线
            }
        }
    },
    watch: {
        value(value) {
            this.val = value
        }
    },
    methods: {
        input() {
            this.$emit('input', this.val)
            this.$emit('update', this.val)
        },
        pressEnter() {
            this.$emit('enter')
        },
        focus() {
            this.$emit('focus')
        },
        keyup() {
            this.regs[this.limit] &&
                (this.val = this.val
                    .toString()
                    .replace(this.regs[this.limit], ''))
            // console.log(this.val)
            this.$emit('keyup', this.val)
            this.$emit('update', this.val)
        }
    },
    mounted() {
        this.val = this.value
    }
}
</script>

<style scoped>
.v-input {
    display: inline-block;
    /* width: 100%; */
    position: relative;
    /* min-height: 26px; */
    /* min-width: 100px; */
    /* background: #fff; */
}
.disabled {
    color: #888;
    cursor: not-allowed;
}
input::placeholder {
    color: #ccc;
}
.large {
    height: 36px;
    line-height: 34px;
}
.normal {
    height: 30px;
    line-height: 28px;
}
.small {
    height: 26px;
    line-height: 26px;
}
/* .small{
        height: 20px;
        line-height: 18px;
    } */
.v-input input {
    width: 100%;
    height: 100%;
    /* border: none; */
    outline: none;
    border-radius: 4px;
    box-sizing: border-box;
    border: 1px solid #ddd;
    padding: 6px 10px;
    background: #fff;
}
.v-input span {
    position: absolute;
    width: 100%;
    height: 100%;
}
.v-input input:focus {
    border-color: #2d8cf0;
}
.v-input .icon-input {
    position: absolute;
    right: 10px;
    color: #808695;
}
.v-input .icon-icon-test {
    cursor: pointer;
}

.large .iconfont {
    font-size: 20px;
}
.normal .iconfont {
    font-size: 18px;
}
.error-message {
    position: absolute;
    top: 25px;
    left: 1em;
    font-size: 12px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    color: #ef583d;
}
.error-message .iconjinggao1- {
    font-size: 13px;
}
</style>>
