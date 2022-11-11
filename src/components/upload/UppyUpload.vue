<script lang="ts" setup>
import { Dashboard } from "@uppy/vue";
import "@uppy/core/dist/style.css";
import "@uppy/dashboard/dist/style.css";
import Uppy, { type SuccessResponse } from "@uppy/core";
import type { Restrictions } from "@uppy/core";
import XHRUpload from "@uppy/xhr-upload";
import zh_CN from "@uppy/locales/lib/zh_CN";
import { computed, onUnmounted } from "vue";

const props = withDefaults(
  defineProps<{
    restrictions?: Restrictions;
    meta?: Record<string, unknown>;
    autoProceed?: boolean;
    allowedMetaFields?: string[];
    endpoint: string;
    name?: string;
    note?: string;
    method?: "GET" | "POST" | "PUT" | "HEAD" | "get" | "post" | "put" | "head";
    disabled?: boolean;
  }>(),
  {
    restrictions: undefined,
    meta: undefined,
    autoProceed: false,
    allowedMetaFields: undefined,
    name: "file",
    note: undefined,
    method: "post",
    disabled: false,
  }
);

const emit = defineEmits<{
  (event: "uploaded", response: SuccessResponse): void;
}>();

const uppy = computed(() => {
  return new Uppy({
    locale: zh_CN,
    meta: props.meta,
    restrictions: props.restrictions,
    autoProceed: props.autoProceed,
  }).use(XHRUpload, {
    endpoint: `${import.meta.env.VITE_API_URL}${props.endpoint}`,
    allowedMetaFields: props.allowedMetaFields,
    withCredentials: true,
    formData: true,
    fieldName: props.name,
    method: props.method,
    limit: 5,
  });
});

uppy.value.on("upload-success", (_, response: SuccessResponse) => {
  emit("uploaded", response);
});

onUnmounted(() => {
  uppy.value.close({ reason: "unmount" });
});
</script>

<template>
  <dashboard
    :uppy="uppy"
    :props="{ theme: 'light', disabled: disabled, note: note }"
  />
</template>