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
      <div class="flex items-center gap-2">
        <HoppButtonSecondary
          v-tippy="{ theme: 'tooltip' }"
          :title="'New chat'"
          :icon="IconSparkles"
          class="rounded hover:bg-primaryDark focus-visible:bg-primaryDark"
          @click="resetChat"
        />
        <HoppButtonSecondary
          v-tippy="{ theme: 'tooltip' }"
          :title="'Close'"
          :icon="IconX"
          class="rounded hover:bg-primaryDark focus-visible:bg-primaryDark"
          @click="invokeAction('modals.chat.toggle')"
        />
      </div>
    </header>
    <section ref="messagesScroller" class="flex-1 min-h-0 overflow-auto p-3">
      <div class="flex flex-col gap-4">
        <template v-for="m in messages" :key="m.id">
          <div
            :class="
              m.role === 'user' ? 'flex justify-end' : 'flex justify-start'
            "
          >
            <div
              class="flex max-w-[85%] items-end gap-2"
              :class="m.role === 'user' ? 'flex-row-reverse' : 'flex-row'"
            >
              <div>
                <HoppSmartPicture v-if="m.role === 'user'" name="you" />
                <div
                  v-else
                  class="flex h-7 w-7 items-center justify-center rounded-full bg-primary"
                >
                  <IconBot class="h-4 w-4 text-secondary" />
                </div>
              </div>
              <div
                class="flex flex-col items-start gap-1"
                :class="m.role === 'user' && 'items-end'"
              >
                <div
                  class="rounded-lg px-3 py-2 whitespace-pre-wrap"
                  :class="
                    m.role === 'user'
                      ? 'bg-accent text-accentContrast'
                      : 'bg-primary text-secondaryDark'
                  "
                >
                  <template v-if="m.kind === 'code'">
                    <div class="mb-2 text-secondary text-tiny">
                      {{ m.text }}
                    </div>
                    <pre
                      class="rounded bg-primaryDark/50 p-2 text-xs overflow-auto"
                    ><code>{{ m.code }}</code></pre>
                  </template>
                  <template v-else>
                    {{ m.text }}
                  </template>
                </div>
                <span class="text-tiny text-secondary">{{ m.time }}</span>
              </div>
            </div>
          </div>
        </template>
      </div>
    </section>
    <footer class="p-3 border-t border-divider/60">
      <div class="flex items-end gap-2">
        <div class="relative">
          <HoppButtonSecondary
            v-tippy="{ theme: 'tooltip' }"
            :title="'Add content'"
            :icon="IconPlusCircle"
            class="rounded hover:bg-primaryDark focus-visible:bg-primaryDark"
            @click="toggleAddMenu"
          />
          <div
            v-if="showAddMenu"
            class="absolute bottom-10 z-10 w-56 rounded border border-divider bg-primaryLight p-2 shadow-lg"
          >
            <div class="text-tiny text-secondary mb-1">Insert</div>
            <HoppButtonSecondary
              :icon="IconFile"
              :label="'Current request details'"
              class="w-full justify-start"
              @click="insertPlaceholder('Add current request context here')"
            />
            <HoppButtonSecondary
              :icon="IconFile"
              :label="'Last response body'"
              class="mt-1 w-full justify-start"
              @click="insertPlaceholder('Add last response body here')"
            />
          </div>
        </div>

        <HoppButtonSecondary
          v-tippy="{ theme: 'tooltip' }"
          :title="'Attach file'"
          :icon="IconFile"
          class="rounded hover:bg-primaryDark focus-visible:bg-primaryDark"
          @click="pickFile"
        />
        <input
          ref="fileInput"
          type="file"
          class="hidden"
          @change="onFilePicked"
        />

        <div class="flex-1">
          <div class="rounded bg-primary p-1">
            <textarea
              v-model="draft"
              class="h-10 max-h-40 w-full resize-none bg-transparent p-2 outline-none"
              placeholder="Message the AI..."
              rows="1"
              @keydown="onKeydown"
              @compositionstart="isComposing = true"
              @compositionend="isComposing = false"
            />
          </div>
          <div class="mt-1 text-tiny text-secondary">
            Enter to send • Shift+Enter for newline
          </div>
        </div>

        <HoppButtonPrimary
          :label="'Send'"
          class="h-9"
          :disabled="!draft.trim()"
          @click="send"
        />
      </div>
    </footer>
  </aside>
</template>

<script setup lang="ts">
import { nextTick, ref, watch } from "vue"
import { invokeAction } from "~/helpers/actions"
import IconBot from "~icons/lucide/bot"
import IconX from "~icons/lucide/x"
import IconPlusCircle from "~icons/lucide/plus-circle"
import IconFile from "~icons/lucide/file"
import IconSparkles from "~icons/lucide/sparkles"

type ChatMessage =
  | { id: string; role: "assistant"; kind: "text"; text: string; time: string }
  | {
      id: string
      role: "assistant"
      kind: "code"
      text: string
      code: string
      time: string
    }
  | { id: string; role: "user"; kind: "text"; text: string; time: string }

const now = () =>
  new Date().toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" })

const messages = ref<ChatMessage[]>([
  {
    id: "m1",
    role: "assistant",
    kind: "text",
    text: "Hi! I’m your AI assistant. How can I help today?",
    time: now(),
  },
  {
    id: "m2",
    role: "user",
    kind: "text",
    text: "Generate a cURL for GET https://api.example.com/users",
    time: now(),
  },
  {
    id: "m3",
    role: "assistant",
    kind: "code",
    text: "Here’s a sample request:",
    code: 'curl -X GET \\\n+  https://api.example.com/users \\\n+  -H "Accept: application/json"',
    time: now(),
  },
])

const draft = ref("")
const isComposing = ref(false)
const showAddMenu = ref(false)
const fileInput = ref<HTMLInputElement | null>(null)
const messagesScroller = ref<HTMLElement | null>(null)

const scrollToBottom = async () => {
  await nextTick()
  const el = messagesScroller.value
  if (!el) return
  el.scrollTop = el.scrollHeight
}

const send = () => {
  const content = draft.value.trim()
  if (!content) return
  messages.value.push({
    id: `${Date.now()}-${Math.random()}`,
    role: "user",
    kind: "text",
    text: content,
    time: now(),
  })
  draft.value = ""
  scrollToBottom()

  // Dummy assistant reply
  setTimeout(() => {
    messages.value.push({
      id: `${Date.now()}-${Math.random()}`,
      role: "assistant",
      kind: "text",
      text: "Got it! (dummy response) — wire me to your agent backend to make this live.",
      time: now(),
    })
    scrollToBottom()
  }, 500)
}

const onKeydown = (e: KeyboardEvent) => {
  if (e.key === "Enter" && !e.shiftKey && !isComposing.value) {
    e.preventDefault()
    send()
  }
}

const toggleAddMenu = () => {
  showAddMenu.value = !showAddMenu.value
}

const insertPlaceholder = (text: string) => {
  draft.value = draft.value ? draft.value + `\n${text}` : text
  showAddMenu.value = false
}

const pickFile = () => fileInput.value?.click()
const onFilePicked = () => {
  // Placeholder for file handling
}

const resetChat = () => {
  messages.value = [
    {
      id: "m1",
      role: "assistant",
      kind: "text",
      text: "Started a new chat. How can I help?",
      time: now(),
    },
  ]
  scrollToBottom()
}

watch(
  () => messages.value.length,
  () => scrollToBottom(),
  { immediate: true }
)
</script>

<style scoped></style>
