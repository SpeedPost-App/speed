<template>
  <aside
    class="flex h-full w-full flex-col border-l border-divider bg-primaryLight"
  >
    <header
      class="flex items-center justify-between px-3 py-2 border-b border-divider/60"
    >
      <div class="flex items-center gap-2">
        <IconBot class="h-4 w-4 text-secondary" />
        <span class="font-semibold">AI Chat</span>
      </div>
      <HoppButtonSecondary
        v-tippy="{ theme: 'tooltip' }"
        :title="'Close'"
        :icon="IconX"
        class="rounded hover:bg-primaryDark focus-visible:bg-primaryDark"
        @click="invokeAction('modals.chat.toggle')"
      />
    </header>
    <section class="flex-1 min-h-0 overflow-auto p-3 space-y-3">
      <div class="text-secondary">
        Chat panel placeholder. Plug your agent here.
      </div>
    </section>
    <footer class="p-3 border-t border-divider/60">
      <div class="flex items-center gap-2">
        <input
          v-model="draft"
          type="text"
          class="flex-1 rounded bg-primary p-2 outline-none focus:ring-1 focus:ring-accent"
          placeholder="Type a message..."
          @keyup.enter="send"
        />
        <HoppButtonPrimary :label="'Send'" class="h-8" @click="send" />
      </div>
    </footer>
  </aside>
</template>

<script setup lang="ts">
import { ref } from "vue"
import { invokeAction } from "~/helpers/actions"
import IconBot from "~icons/lucide/bot"
import IconX from "~icons/lucide/x"

const draft = ref("")

const send = () => {
  // Placeholder action; wire to your agent later
  draft.value = ""
}
</script>

<style scoped></style>
