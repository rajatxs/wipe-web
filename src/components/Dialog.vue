<script setup>
import { ref, onMounted } from 'vue';
import IconButton from './IconButton.vue';
import CloseIcon from '../icons/close.vue';

const dialogRef = ref(null);
const emit = defineEmits(['close']);
const props = defineProps({
    title: {
        type: String,
        required: true,
    }
});

onMounted(function () {
    if (dialogRef.value instanceof HTMLElement) {
        dialogRef.value.showModal();
    }
});
</script>

<template>
    <dialog ref="dialogRef" class="app-dialog">
        <header class="app-dialog__header">
            <h5>{{ props.title }}</h5>
            <IconButton @click="emit('close')">
                <CloseIcon class="w-5 h-5" />
            </IconButton>
        </header>
        <main class="app-dialog__main">
            <slot></slot>
        </main>
    </dialog>
</template>

<style>
.app-dialog {
    @apply relative w-11/12 sm:w-1/2 sm:max-w-[380px] z-50 outline-none rounded-2xl overflow-hidden bg-white dark:bg-neutral-900 dark:text-white;
}
.app-dialog::backdrop {
    background-color: rgba(0, 0, 0, 0.5);
}
.app-dialog__header {
    @apply sticky flex justify-between items-center px-5 h-16 font-bold border-b dark:border-b-neutral-700;
}
.app-dialog__main {
    @apply p-5 overflow-hidden;
}
</style>
