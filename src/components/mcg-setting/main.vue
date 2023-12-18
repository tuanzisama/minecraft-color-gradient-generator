<template>
  <t-drawer
    v-model:visible="privateValue"
    header="偏好设置"
    size="600px"
    :close-on-overlay-click="false"
    :close-on-esc-keydown="false"
    @confirm="onConfirmHandler"
  >
    <t-form class="setting-form" :data="formData" label-width="120px">
      <t-form-item label="字符模式" name="charMode">
        <t-radio-group v-model="formData.charMode">
          <t-radio value="&">&</t-radio>
          <t-radio value="§">§</t-radio>
        </t-radio-group>
      </t-form-item>
      <t-form-item label="兼容模式" name="compatibleMode" class="warped-formitem">
        <t-switch v-model="formData.compatibleMode" />
        <p class="setting-item-summary">仅限服务器不支持&amp;&num;前缀发送Hex颜色</p>
      </t-form-item>
      <t-form-item label="移除空格" name="clearSpaceCharacter" class="warped-formitem">
        <t-switch v-model="formData.clearSpaceCharacter" />
        <p class="setting-item-summary">移除空格/换行/制表符</p>
      </t-form-item>
      <t-divider>TDesign</t-divider>
      <t-form-item label="清空渐变色列表">
        <t-popconfirm content="确认清空吗" @confirm="onResetClickHandler">
          <t-button theme="danger" :disabled="colorStore.cacheColorList.length === 0">
            清空 {{ colorStore.cacheColorList.length }} 个渐变色
          </t-button>
        </t-popconfirm>
      </t-form-item>
    </t-form>
  </t-drawer>
</template>

<script lang="ts" setup>
import { computed, reactive, watch } from "vue";
import { AppStoreState, useAppStore } from "../../plugins/store/modules/app";
import { useColorStore } from "../../plugins/store/modules/color";
import { MessagePlugin } from "tdesign-vue-next";

const props = withDefaults(defineProps<McgSettingProps>(), {
  modelValue: false,
});
const emit = defineEmits<McgSettingEmit>();
const appStore = useAppStore();
const colorStore = useColorStore();
const formData = reactive<AppStoreState["setting"]>({
  charMode: "&",
  compatibleMode: false,
  clearSpaceCharacter: true,
});

watch(
  () => props.modelValue,
  (newVal) => {
    if (newVal) {
      formData.charMode = appStore.setting.charMode;
      formData.compatibleMode = appStore.setting.compatibleMode;
      formData.clearSpaceCharacter = appStore.setting.clearSpaceCharacter;
    }
  }
);

const privateValue = computed({
  get: () => props.modelValue,
  set: (val) => emit("update:modelValue", val),
});

const onConfirmHandler = () => {
  appStore.saveSetting(formData);
  privateValue.value = false;
  MessagePlugin.success({ content: "已保存", placement: "bottom" });
};

const onResetClickHandler = () => {
  colorStore.resetCacheColorList();
  privateValue.value = false;
  MessagePlugin.success({ content: "已清空", placement: "bottom" });
};
</script>

<script lang="ts">
export interface McgSettingProps {
  modelValue: boolean;
}
export interface McgSettingEmit {
  (e: "update:modelValue", value: McgSettingProps["modelValue"]): void;
}
</script>

<style lang="scss" scoped>
.setting-form {
  .setting-item-summary {
    flex-basis: 100%;
    margin-top: 10px;
    padding: 5px 10px;
    background: #f5f7fa;
    border: 1px solid #dcdfe6;
    border-radius: 5px;
  }

  .warped-formitem:deep(.t-form__controls-content) {
    flex-wrap: wrap;
  }
}
</style>