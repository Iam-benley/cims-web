<template>
  <!-- Backdrop -->
  <div
    v-if="show"
    class="modal-backdrop fade"
    :class="{ show: show }"
    @click="closeOnBackdrop && close()"
  ></div>

  <!-- Dialog -->
  <div
    v-if="show"
    class="modal fade"
    :class="{ show: show }"
    tabindex="-1"
    :style="{ display: 'block' }"
  >
    <div class="modal-dialog" :class="modalSizeClass">
      <div class="modal-content">
        <!-- Header -->
        <div class="modal-header">
          <h5 class="modal-title">{{ title }}</h5>
          <button type="button" class="btn-close" @click="close"></button>
        </div>

        <!-- Body -->
        <div class="modal-body">
          <slot name="body">
            <p>{{ message }}</p>
          </slot>
        </div>

        <!-- Footer -->
        <div class="modal-footer">
          <slot name="footer"> </slot>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from "vue";

const props = defineProps({
  show: {
    type: Boolean,
    required: true,
  },
  title: {
    type: String,
    default: "Dialog Title",
  },
  message: {
    type: String,
    default: "Default dialog message.",
  },
  size: {
    type: String,
    default: "md", // sm, md, lg, xl
    validator: (value) => ["sm", "md", "lg", "xl"].includes(value),
  },
  closeOnBackdrop: {
    type: Boolean,
    default: true,
  },
});

const emit = defineEmits(["update:show", "confirm", "close"]);

const modalSizeClass = computed(() => {
  if (props.size === "sm") return "modal-sm";
  if (props.size === "lg") return "modal-lg";
  if (props.size === "xl") return "modal-xl";
  return "";
});

const close = () => {
  emit("update:show", false);
  emit("close");
};

const confirm = () => {
  emit("confirm");
  close();
};
</script>

<style scoped>
.modal {
  background-color: rgba(0, 0, 0, 0.5);
}

.modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 1040;
  width: 100vw;
  height: 100vh;
  background-color: #000;
  opacity: 0;
}

.modal-backdrop.show {
  opacity: 0.5;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 1050;
  width: 100%;
  height: 100%;
  overflow-x: hidden;
  overflow-y: auto;
  outline: 0;
  opacity: 0;
  pointer-events: none;
}

.modal.show {
  opacity: 1;
  pointer-events: auto;
}

.modal-dialog {
  position: relative;
  width: auto;
  margin: 1.75rem auto;
  pointer-events: none;
}

.modal-content {
  position: relative;
  display: flex;
  flex-direction: column;
  width: 100%;
  pointer-events: auto;
  background-color: #fff;
  background-clip: padding-box;
  border: 1px solid rgba(0, 0, 0, 0.2);
  border-radius: 0.3rem;
  outline: 0;
}
</style>
