<template>
  <view :class="classes">
    <view
      class="nut-rate-item"
      v-for="n in count"
      :key="n"
      :style="{ marginRight: pxCheck(spacing) }"
    >
      <nut-icon
        :size="iconSize"
        class="nut-rate-item__icon"
        @click="onClick(1, n)"
        :class="{ 'nut-rate-item__icon--disabled': disabled || n > modelValue }"
        :color="n <= modelValue ? activeColor : voidColor"
        :name="n <= modelValue ? checkedIcon : uncheckedIcon"
      />
      <nut-icon
        v-if="allowHalf && modelValue + 1 > n"
        class="nut-rate-item__icon nut-rate-item__icon--half"
        @click="onClick(2, n)"
        :color="n <= modelValue ? activeColor : voidColor"
        :size="iconSize"
        :name="checkedIcon"
      />
      <nut-icon
        v-else-if="allowHalf && modelValue + 1 < n"
        class="
          nut-rate-item__icon
          nut-rate-item__icon--disabled
          nut-rate-item__icon--half
        "
        @click="onClick(2, n)"
        :color="voidColor"
        :size="iconSize"
        :name="uncheckedIcon"
      />
    </view>
  </view>
</template>
<script lang="ts">
import { computed } from 'vue';
import { createComponent } from '../../utils/create';
import { pxCheck } from '../../utils/pxCheck';
const { componentName, create } = createComponent('rate');
import Taro from '@tarojs/taro';
export default create({
  props: {
    count: {
      type: [String, Number],
      default: 5
    },
    modelValue: {
      type: [String, Number],
      default: 0
    },
    iconSize: {
      type: [String, Number],
      default: 18
    },
    activeColor: {
      type: String,
      default: ''
    },
    voidColor: {
      type: String,
      default: ''
    },
    uncheckedIcon: {
      type: String,
      default: 'star-n'
    },
    checkedIcon: {
      type: String,
      default: 'star-fill-n'
    },
    readonly: {
      type: Boolean,
      default: false
    },
    disabled: {
      type: Boolean,
      default: false
    },
    allowHalf: {
      type: Boolean,
      default: false
    },
    spacing: {
      type: [String, Number],
      default: 14
    }
  },
  emits: ['update:modelValue', 'change'],
  setup(props, { emit }) {
    const classes = computed(() => {
      const prefixCls = componentName;
      return {
        [prefixCls]: true
      };
    });
    const onClick = (e: number, index: number) => {
      if (props.disabled || props.readonly) return;
      let value = 0;
      if (index === 1 && props.modelValue === index) {
      } else {
        value = index;
        if (props.allowHalf && e == 2) {
          value -= 0.5;
        }
      }
      emit('update:modelValue', value);
      emit('change', value);
    };

    return {
      classes,
      onClick,
      pxCheck
    };
  }
});
</script>
