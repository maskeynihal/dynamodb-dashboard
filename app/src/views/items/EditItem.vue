<template>
  <div class="row">
    <div
      class="col-12 col-xl-8"
      id="edit-item"
    >
      <h3>Edit item</h3>
      <p>
        You can add, remove, or edit the attributes of an item. You can nest attributes inside other attributes up to 32
        levels deep.
      </p>

      <div class="vh-60">
        <textarea ref="textAreaRef" />
      </div>

      <div
        class="alert alert-danger rounded-0"
        role="alert"
        v-if="!!errorMsg"
      >
        {{ errorMsg }}
      </div>

      <div
        class="alert alert-primary rounded-0"
        role="alert"
        v-if="hasKeyChanged"
      >
        You are changing the values of the item keys. A transaction will be used to delete and recreate the item with
        the new keys.
      </div>

      <br />
      <br />
      <br />

      <div class="d-flex justify-content-end">
        <button
          class="btn btn-primary rounded-0"
          type="button"
          :disabled="!isValid || store.ui.state.isLoading"
          @click="save"
        >
          <span
            v-if="store.ui.state.isLoading"
            class="spinner-grow spinner-grow-sm"
            role="status"
            aria-hidden="true"
          ></span>
          <span class="visually-hidden">Loading...</span>
          Save Changes
        </button>

        <button
          type="button"
          class="btn btn-secondary rounded-0 ms-2"
          @click="cancel"
        >
          Cancel
        </button>
      </div>

      <!-- Toast -->
      <div class="toast-container position-fixed top-0 start-50 translate-middle-x p-3">
        <div
          class="toast align-items-center border-0"
          :class="toast.className"
          role="alert"
          aria-live="assertive"
          aria-atomic="true"
          ref="toastRef"
        >
          <div class="d-flex">
            <div class="toast-body">{{ toast.message }}</div>
            <button
              type="button"
              class="btn-close btn-close-white me-2 m-auto"
              data-bs-dismiss="toast"
              aria-label="Close"
            ></button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
  import * as bootstrap from "bootstrap";
  import { useRoute, useRouter } from "vue-router";
  import { inject, onMounted, reactive, ref, watch } from "vue";

  import { getTable } from "@/services/table";
  import codeMirrorConfig from "./codeMirrorConfig";
  import { getItem, updateItem } from "@/services/item";

  export default {
    setup() {
      let codeMirror;
      const item = ref("{}");
      const editItem = ref("{}");
      const errorMsg = ref("");

      const isValid = ref(true);
      const hasKeyChanged = ref(false);

      const toast = reactive({
        className: "text-bg-success",
        message: "The item has been saved successfully.",
      });

      const route = useRoute();
      const router = useRouter();
      const textAreaRef = ref(null);
      const toastRef = ref(null);
      const store = inject("store");

      const cancel = () => {
        if (window.history.length > 1) {
          router.back();
        } else {
          router.push({ name: "home" });
        }
      };

      const save = async () => {
        const table = store.table.state.Table;
        const tableName = table.TableName;

        try {
          const { ref, body } = await updateItem(
            tableName,
            JSON.parse(item.value),
            JSON.parse(editItem.value),
            table.KeySchema,
          );

          toast.className = "text-bg-success";
          toast.message = "The item has been saved successfully.";

          if (hasKeyChanged.value) {
            item.value = JSON.stringify(body, null, 2);
            router.replace({
              name: "edit-item",
              query: { ...route.query, ...ref },
            });
          }
        } catch (error) {
          toast.className = "text-bg-danger";
          toast.message = error.message;
        } finally {
          const toastEl = new bootstrap.Toast(toastRef.value, { delay: 1000 });
          setTimeout(() => toastEl.show(), 0);
        }
      };

      watch([item, editItem], ([original, edited]) => {
        isValid.value = false;
        errorMsg.value = "";

        try {
          const validItem = JSON.parse(edited);
          const originalItem = JSON.parse(original);
          isValid.value = true;
          hasKeyChanged.value = false;

          Object.keys(route.query).forEach((key) => {
            hasKeyChanged.value ||= originalItem[key] !== validItem[key];
          });
        } catch (error) {
          errorMsg.value = error.message;
          isValid.value = false;
        }
      });

      // https://codemirror.net/
      onMounted(async () => {
        codeMirror = codeMirrorConfig(textAreaRef, editItem);

        await initItem();

        codeMirror.dispatch({
          changes: {
            from: 0,
            to: codeMirror.state.doc.length,
            insert: item.value,
          },
        });
      });

      const initItem = async () => {
        const { tableName } = route.params;
        const table = await getTable(tableName);

        const { Item } = await getItem(tableName, route.query, table.AttributeDefinitions);

        item.value = JSON.stringify(Item, null, 2);
        store.table.setters.setTable(table);
      };

      return {
        store,
        textAreaRef,
        isValid,
        hasKeyChanged,
        toast,
        toastRef,
        cancel,
        save,
        errorMsg,
      };
    },
  };
</script>
