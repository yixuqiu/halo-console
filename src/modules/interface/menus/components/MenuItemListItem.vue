<script lang="ts" setup>
import {
  IconList,
  IconSettings,
  VButton,
  VSpace,
  VTag,
} from "@halo-dev/components";
import Draggable from "vuedraggable";
import { ref } from "vue";
import type { MenuTreeItem } from "@/modules/interface/menus/utils";

withDefaults(
  defineProps<{
    menuTreeItems: MenuTreeItem[];
  }>(),
  {
    menuTreeItems: () => [],
  }
);

const emit = defineEmits<{
  (event: "change"): void;
  (event: "open-editing", menuItem: MenuTreeItem): void;
  (event: "delete", menuItem: MenuTreeItem): void;
}>();

const isDragging = ref(false);

function onChange() {
  emit("change");
}

function onOpenEditingModal(menuItem: MenuTreeItem) {
  emit("open-editing", menuItem);
}

function onDelete(menuItem: MenuTreeItem) {
  emit("delete", menuItem);
}

function getMenuItemRefDisplayName(menuItem: MenuTreeItem) {
  if (menuItem.spec.postRef) {
    return "文章";
  }
  if (menuItem.spec.singlePageRef) {
    return "自定义页面";
  }
  if (menuItem.spec.categoryRef) {
    return "分类";
  }
  if (menuItem.spec.tagRef) {
    return "标签";
  }
  return undefined;
}
</script>
<template>
  <draggable
    :list="menuTreeItems"
    class="box-border h-full w-full divide-y divide-gray-100"
    ghost-class="opacity-50"
    group="menu-item"
    handle=".drag-element"
    item-key="metadata.name"
    tag="ul"
    @change="onChange"
    @end="isDragging = false"
    @start="isDragging = true"
  >
    <template #item="{ element: menuItem }">
      <li>
        <div
          class="group relative block cursor-pointer px-4 py-3 transition-all hover:bg-gray-50"
        >
          <div
            class="drag-element absolute inset-y-0 left-0 hidden w-3.5 cursor-move items-center bg-gray-100 transition-all hover:bg-gray-200 group-hover:flex"
          >
            <IconList class="h-3.5 w-3.5" />
          </div>
          <div class="relative flex flex-row items-center">
            <div class="flex-1">
              <div class="flex flex-row items-center gap-2">
                <span class="truncate text-sm font-medium text-gray-900">
                  {{ menuItem.status.displayName }}
                </span>
                <VTag v-if="getMenuItemRefDisplayName(menuItem)">
                  {{ getMenuItemRefDisplayName(menuItem) }}
                </VTag>
              </div>

              <div class="mt-1 flex">
                <VSpace align="start" direction="column" spacing="xs">
                  <a
                    :href="menuItem.status.href"
                    class="text-xs text-gray-500 hover:text-gray-900"
                    target="_blank"
                  >
                    {{ menuItem.status.href }}
                  </a>
                </VSpace>
              </div>
            </div>
            <FloatingTooltip
              v-if="menuItem.metadata.deletionTimestamp"
              class="mr-4 hidden items-center sm:flex"
            >
              <div class="inline-flex h-1.5 w-1.5 rounded-full bg-red-600">
                <span
                  class="inline-block h-1.5 w-1.5 animate-ping rounded-full bg-red-600"
                ></span>
              </div>
              <template #popper> 删除中</template>
            </FloatingTooltip>
            <div class="self-center">
              <FloatingDropdown>
                <IconSettings
                  class="cursor-pointer transition-all hover:text-blue-600"
                />
                <template #popper>
                  <div class="w-48 p-2">
                    <VSpace class="w-full" direction="column">
                      <VButton
                        v-close-popper
                        block
                        type="secondary"
                        @click="onOpenEditingModal(menuItem)"
                      >
                        修改
                      </VButton>
                      <VButton
                        v-close-popper
                        block
                        type="danger"
                        @click="onDelete(menuItem)"
                      >
                        删除
                      </VButton>
                    </VSpace>
                  </div>
                </template>
              </FloatingDropdown>
            </div>
          </div>
        </div>

        <MenuItemListItem
          :menu-tree-items="menuItem.spec.children"
          class="pl-10 transition-all duration-300"
          @change="onChange"
          @delete="onDelete"
          @open-editing="onOpenEditingModal"
        />
      </li>
    </template>
  </draggable>
</template>