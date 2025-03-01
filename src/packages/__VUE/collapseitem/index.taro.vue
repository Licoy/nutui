<template>
  <view :class="classes">
    <view
      :class="['collapse-item', { 'item-expanded': openExpanded }, { 'nut-collapse-item-disabled': disabled }]"
      @click="toggleOpen"
    >
      <view class="collapse-title">
        <view>
          <view class="collapse-title-value">
            <nut-icon
              v-if="titleIcon"
              :name="titleIcon"
              :size="titleIconSize"
              :color="titleIconColor"
              :class="['collapse-title-icon', titleIconPosition == 'left' ? 'titleIconLeft' : 'titleIconRight']"
            ></nut-icon>
            <template v-if="$slots.mTitle">
              <slot name="mTitle"></slot>
            </template>
            <template v-else>
              <view v-html="title" class="collapse-icon-title"></view>
            </template>
            <!-- <view
              v-html="title"
              class="collapse-icon-title"
              v-if="title"
            ></view>
            <mTitle v-else>
              <slot name="mTitle"></slot>
            </mTitle> -->
          </view>
        </view>
      </view>
      <!-- <view v-if="subTitle" v-html="subTitle" class="subTitle"></view>
      <view class="subTitle" v-else>
        <slot name="sTitle"></slot>
      </view> -->
      <view v-if="$slots.sTitle" class="subTitle">
        <slot name="sTitle"></slot>
      </view>
      <view v-else v-html="subTitle" class="subTitle"></view>
      <nut-icon
        v-if="icon"
        :name="icon"
        :size="iconSize"
        :color="iconColor"
        :class="['collapse-icon', { 'col-expanded': openExpanded }, { 'collapse-icon-disabled': disabled }]"
        :style="iconStyle"
      ></nut-icon>
    </view>
    <view
      :class="['collapse-wrapper', openExpanded ? 'open-style' : 'close-style']"
      ref="wrapperRef"
      :style="{ height: openExpanded ? conHeight + 'px' : 0 }"
    >
      <view class="collapse-content" ref="contentRef">
        <slot></slot>
      </view>
    </view>
  </view>
</template>
<script lang="ts">
import {
  reactive,
  inject,
  toRefs,
  onMounted,
  Ref,
  ref,
  unref,
  nextTick,
  computed,
  watch,
  getCurrentInstance,
  ComponentInternalInstance
} from 'vue';
import Taro, { eventCenter, getCurrentInstance as getCurrentInstanceTaro } from '@tarojs/taro';
import { createComponent } from '../../utils/create';
import { useTaroRect } from '../../utils/useTaroRect';
const { create, componentName } = createComponent('collapse-item');

export default create({
  props: {
    title: {
      type: String,
      default: ''
    },
    subTitle: {
      type: String,
      default: ''
    },
    disabled: {
      type: Boolean,
      default: false
    },
    name: {
      type: [Number, String],
      default: -1,
      required: true
    },
    collapseRef: {
      type: Object
    }
  },
  setup(props) {
    const collapse: any = inject('collapseParent');
    const conHeight: any = ref(0);
    const parent: any = reactive(collapse);
    const classes = computed(() => {
      const prefixCls = componentName;
      return {
        [prefixCls]: true,
        // [`${prefixCls}-left`]: parent.props.classDirection === 'left',
        [`${prefixCls}-icon`]: parent.props.icon
      };
    });

    const relation = (child: ComponentInternalInstance): void => {
      if (child.proxy) {
        parent.children.push(child.proxy);
      }
    };
    relation(getCurrentInstance() as ComponentInternalInstance);
    const proxyData: any = reactive({
      icon: parent.props.icon,
      iconSize: parent.props.iconSize,
      iconColor: parent.props.iconColor,
      openExpanded: null,
      // classDirection: 'right',
      iconStyle: {
        transform: 'rotate(0deg)',
        marginTop: parent.props.iconHeght ? '-' + parent.props.iconHeght / 2 + 'px' : '-10px'
      }
    });

    const titleIconStyle = reactive({
      titleIcon: parent.props.titleIcon,
      titleIconSize: parent.props.titleIconSize,
      titleIconColor: parent.props.titleIconColor,
      titleIconPosition: parent.props.titleIconPosition
      // titleIconWH: {
      //   width: '13px',
      //   height: '13px'
      // }
    });

    // 获取 Dom 元素
    const wrapperRef: any = ref(null);
    const contentRef: any = ref(null);

    // 清除 willChange 减少性能浪费
    const onTransitionEnd = () => {
      nextTick(() => {
        parent.children.forEach((item1: any, index: number) => {
          let ary = Array.from(item1.$el.children);
          ary.forEach((item2: any, index: number) => {
            if (item2.className.includes('collapse-wrapper')) {
              item2.style.willChange = 'auto';
            }
          });
        });
      });
    };

    // 手风琴模式
    const animation = () => {
      if (parent.props.icon && !proxyData.openExpanded) {
        proxyData.iconStyle['transform'] = 'rotate(0deg)';
      } else {
        proxyData.iconStyle['transform'] = 'rotate(' + parent.props.rotate + 'deg)';
      }
      nextTick(() => {
        const query = Taro.createSelectorQuery();
        query.selectAll('.collapse-content').boundingClientRect();
        query.exec((res) => {
          getH(res[0]);
        });
        if (!proxyData.openExpanded) {
          onTransitionEnd();
        }
      });
    };

    const open = () => {
      proxyData.openExpanded = !proxyData.openExpanded;
      animation();
    };

    const defaultOpen = () => {
      open();
      if (parent.props.icon) {
        proxyData['iconStyle']['transform'] = 'rotate(' + parent.props.rotate + 'deg)';
      }
    };

    const currentName = computed(() => props.name);
    const toggleOpen = () => {
      if (parent.props.accordion) {
        parent.children.forEach((item: any, index: number) => {
          if (currentName.value == item.name) {
            item.changeOpen(!item.openExpanded);
          } else {
            item.changeOpen(false);
            item.animation();
          }
        });
        nextTick(() => {
          parent.changeVal(currentName.value);
          animation();
        });
      } else {
        parent.changeValAry(props.name);
        open();
      }
    };
    // 更改子组件展示
    const changeOpen = (bol: boolean) => {
      proxyData.openExpanded = bol;
    };

    const expanded = computed(() => {
      if (parent) {
        return parent.isExpanded(props.name);
      }
      return null;
    });

    watch(expanded, (value, oldValue) => {
      if (value) {
        proxyData.openExpanded = true;
      }
    });

    const getH = (list: any) => {
      parent.children.forEach((item1: any, index1: number) => {
        let ary: any = Array.from(item1.$el.children);
        let _uid = ary[1].children[0]['uid'];
        let tm = list.filter((item2: any) => item2.id == _uid);
        if (tm && tm.length > 0) {
          let h = tm[0]['height'];
          item1.conHeight = h;
        }
      });
    };

    const getH5 = () => {
      parent.children.forEach((item1: any, index1: number) => {
        let ary: any = Array.from(item1.$el.children);
        let h = ary[1].children[0]['offsetHeight'];
        item1.conHeight = h;
      });
    };

    const getRefHeight = () => {
      const query = Taro.createSelectorQuery();
      query.selectAll('.collapse-content').boundingClientRect();
      query.exec((res) => {
        if (Taro.getEnv() === 'WEB') {
          getH5();
        } else {
          getH(res[0]);
        }
      });
    };

    onMounted(() => {
      const { name } = props;
      const active = parent && parent.props.active;
      // 获取 DOM 元素
      if (Taro.getEnv() === 'WEB') {
        getRefHeight();
      } else {
        eventCenter.once((getCurrentInstanceTaro() as any).router.onReady, () => {
          getRefHeight();
        });
      }

      // const query = Taro.createSelectorQuery();
      // query.selectAll('.collapse-content').boundingClientRect();
      // query.exec((res) => {
      //   console.log(res[0]);
      //   getH(res[0]);
      // });

      if (typeof active == 'number' || typeof active == 'string') {
        if (name == active) {
          defaultOpen();
        }
      } else if (Object.values(active) instanceof Array) {
        const f = Object.values(active).filter((item) => item == name);
        if (f.length > 0) {
          defaultOpen();
        }
      }
    });

    return {
      classes,
      ...toRefs(proxyData),
      ...toRefs(parent.props),
      ...toRefs(titleIconStyle),
      conHeight,
      wrapperRef,
      contentRef,
      open,
      toggleOpen,
      changeOpen,
      animation
    };
  }
});
</script>
