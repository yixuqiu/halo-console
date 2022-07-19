<script lang="ts" name="UserCreationModal" setup>
import type { PropType } from "vue";
import { computed, onMounted, ref, watch } from "vue";
import { apiClient } from "@halo-dev/admin-shared";
import type { Role, User } from "@halo-dev/api-client";
import { IconSave, VButton, VModal } from "@halo-dev/components";
import { v4 as uuid } from "uuid";

const props = defineProps({
  visible: {
    type: Boolean,
    default: false,
  },
  user: {
    type: Object as PropType<User | null>,
    default: null,
  },
});

const emit = defineEmits(["update:visible", "close"]);

interface EditingFormState {
  user: User;
  saving: boolean;
}

const roles = ref<Role[]>([]);
const editingFormState = ref<EditingFormState>({
  user: {
    spec: {
      displayName: "",
      avatar: "",
      email: "",
      phone: "",
      password: "",
      bio: "",
      disabled: false,
      loginHistoryLimit: 0,
    },
    apiVersion: "v1alpha1",
    kind: "User",
    metadata: {
      name: uuid(),
    },
  },
  saving: false,
});
const selectedRole = ref("");

const isUpdateMode = computed(() => {
  return !!editingFormState.value.user.metadata.creationTimestamp;
});

const creationModalTitle = computed(() => {
  return isUpdateMode.value ? "编辑用户" : "新增用户";
});

const basicRoles = computed(() => {
  return roles.value.filter(
    (role) =>
      role.metadata?.labels?.["plugin.halo.run/role-template"] !== "true"
  );
});

watch(props, (newVal) => {
  if (newVal.visible && props.user) {
    editingFormState.value.user = props.user;
  }
});

const handleFetchRoles = async () => {
  try {
    const { data } = await apiClient.extension.role.listv1alpha1Role();
    roles.value = data.items;
  } catch (e) {
    console.error(e);
  }
};

const handleVisibleChange = (visible: boolean) => {
  emit("update:visible", visible);
  if (!visible) {
    emit("close");
  }
};

const handleCreateUser = async () => {
  try {
    editingFormState.value.saving = true;
    let user: User;

    if (isUpdateMode.value) {
      const response = await apiClient.extension.user.updatev1alpha1User(
        editingFormState.value.user.metadata.name,
        editingFormState.value.user
      );
      user = response.data;
    } else {
      const response = await apiClient.extension.user.createv1alpha1User(
        editingFormState.value.user
      );
      user = response.data;
    }

    if (selectedRole.value) {
      await apiClient.user.grantPermission(user.metadata.name, {
        // @ts-ignore
        roles: [selectedRole.value],
      });
    }

    handleVisibleChange(false);
  } catch (e) {
    console.error(e);
  } finally {
    editingFormState.value.saving = false;
  }
};

onMounted(handleFetchRoles);
</script>
<template>
  <VModal
    :title="creationModalTitle"
    :visible="visible"
    :width="700"
    @update:visible="handleVisibleChange"
  >
    <FormKit id="user-form" type="form" @submit="handleCreateUser">
      <FormKit
        v-model="editingFormState.user.metadata.name"
        :disabled="true"
        label="用户名"
        type="text"
        validation="required"
      ></FormKit>
      <FormKit
        v-model="editingFormState.user.spec.displayName"
        label="显示名称"
        type="text"
        validation="required"
      ></FormKit>
      <FormKit
        v-model="editingFormState.user.spec.email"
        label="电子邮箱"
        type="email"
        validation="required"
      ></FormKit>
      <FormKit
        v-model="selectedRole"
        :options="
          basicRoles.map((role:Role) => {
            return {
              label: role.metadata?.annotations?.['plugin.halo.run/display-name'] || role.metadata.name,
              value: role.metadata?.name,
            };
          })
        "
        label="角色"
        type="select"
        validation="required"
      ></FormKit>
      <FormKit
        v-model="editingFormState.user.spec.phone"
        label="手机号"
        type="text"
      ></FormKit>
      <FormKit
        v-model="editingFormState.user.spec.avatar"
        label="头像"
        type="text"
      ></FormKit>
      <FormKit
        v-model="editingFormState.user.spec.bio"
        label="描述"
        type="textarea"
      ></FormKit>
    </FormKit>
    <template #footer>
      <VButton
        :loading="editingFormState.saving"
        type="secondary"
        @click="$formkit.submit('user-form')"
      >
        <template #icon>
          <IconSave class="h-full w-full" />
        </template>
        保存
      </VButton>
    </template>
  </VModal>
</template>