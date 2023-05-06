<script setup>
import { ref, computed, onMounted } from "vue"
import { Head } from "@inertiajs/vue3"
import ChatLayout from "@/Layouts/ChatLayout.vue"
import { useForm, Link } from "@inertiajs/vue3"
import ChatContent from "@/Components/ChatContent.vue"
import Skeletron from "@/Components/Skeletron.vue"

const promptInput = ref(null)
const chatContainer = ref(null)
const showDeleteButton = ref(false)

const props = defineProps({
    messages: Array,
    chat: null | Object,
})

const form = useForm({
    prompt: ""
})

const submit = () => {
    const url = props.chat ? `/chat/${props.chat?.id}` : '/chat'
    form.post(url, {
        onFinish: () => clear(),
    })
}

const scrollToBottom = () => {
    if(props.chat) {
        const el = chatContainer.value
        el.scrollTop = el.scrollHeight;
    }
}

const clear = () => {
    form.prompt = ""
    promptInput.value.focus()
    scrollToBottom()
}

const title = computed(() => props.chat?.context[0].content ?? "New Chat")

onMounted(() => {
    clear()
})

</script>

<template>
    <Head :title="title" />
    <ChatLayout>
        <template #aside>
            <ul class="p-2">
                <li class="px-4 py-2 my-2 flex justify-between font-semibold text-blue-400 bg-slate-900 hover:bg-slate-700 rounded-lg duration-200">
                    <Link href="/" class="w-full">Go Home</Link>
                    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M2.25 12l8.954-8.955c.44-.439 1.152-.439 1.591 0L21.75 12M4.5 9.75v10.125c0 .621.504 1.125 1.125 1.125H9.75v-4.875c0-.621.504-1.125 1.125-1.125h2.25c.621 0 1.125.504 1.125 1.125V21h4.125c.621 0 1.125-.504 1.125-1.125V9.75M8.25 21h8.25" />
                    </svg>
                </li>
                <li v-if="chat" class="px-4 py-2 my-2 flex justify-between font-semibold text-green-400 bg-slate-900 hover:bg-slate-700 rounded-lg duration-200">
                    <Link href="/chat/" class="w-full">New Chat</Link>
                    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M12 4.5v15m7.5-7.5h-15" />
                    </svg>
                </li>
                <template v-for="message in messages" :key="message.id">
                    <li :class="[message.id === chat?.id ? 'bg-slate-700' : 'bg-slate-900', 'px-4 py-2 my-2 flex justify-between font-semibold text-slate-400 hover:bg-slate-700 rounded-lg duration-200']">
                        <Link :href="`/chat/${message.id}`">{{ message.context[0].content }}</Link>
                        <div v-if="message.id === chat?.id">
                            <button v-if="!showDeleteButton" @click="showDeleteButton = !showDeleteButton">
                                <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6 text-red-300">
                                    <path stroke-linecap="round" stroke-linejoin="round"
                                        d="M14.74 9l-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 01-2.244 2.077H8.084a2.25 2.25 0 01-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 00-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 013.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 00-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 00-7.5 0"
                                    />
                                </svg>
                            </button>
                            <span v-if="showDeleteButton" class="flex justify-between">
                                <Link :href="route('chat.destroy', chat?.id)" method="DELETE" as="button" class="text-red-300 hover:text-red-500">
                                    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
                                        <path stroke-linecap="round" stroke-linejoin="round" d="M4.5 12.75l6 6 9-13.5" />
                                    </svg>
                                </Link>
                                <button @click="showDeleteButton = false" class="text-slate-300 hover:text-slate-500">
                                    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
                                        <path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
                                    </svg>
                                </button>
                            </span>
                        </div>
                    </li>
                </template>
            </ul>
        </template>
        <div class="w-full flex text-white">
            <template v-if="chat">
                <div class="w-full flex h-screen bg-slate-900">
                    <div class="w-full overflow-auto pb-36" ref="chatContainer">
                        <template
                            v-for="(content, index) in chat?.context"
                            :key="index"
                        >
                            <ChatContent :content="content" />
                        </template>
                        <Skeleton v-show="form.processing" />
                    </div>
                </div>
            </template>
        </div>
        <template #form>
            <section class="px-6 top-0">
                <div class="w-full">
                    <div class="relative flex-1 flex items-center">
                        <input type="text" class="w-full bg-slate-700 text-white rounded-lg"
                        placeholder="Faça uma pergunta para a OpenAI" v-model="form.prompt" @keyup.enter="submit" :disabled="form.processing" ref="promptInput" />
                        <div class="absolute inset-y-0 right-0 flex items-center pl-3">
                            <svg v-if="!form.processing" @click="submit" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6 -ml-8 text-slate-200 cursor-pointer">
                                <path stroke-linecap="round" stroke-linejoin="round" d="M6 12L3.269 3.126A59.768 59.768 0 0121.485 12 59.77 59.77 0 013.27 20.876L5.999 12zm0 0h7.5"/>
                            </svg>
                            <div class="dot-typing -ml-10" v-if="form.processing"></div>
                        </div>
                    </div>
                </div>
            </section>
        </template>
    </ChatLayout>
</template>

<style>
.dot-typing {
    position: relative;
    left: -9999px;
    width: 10px;
    height: 10px;
    border-radius: 5px;
    background-color: #9880ff;
    color: #9880ff;
    box-shadow: 9984px 0 0 0 #9880ff, 9999px 0 0 0 #9880ff,
        10014px 0 0 0 #9880ff;
    animation: dot-typing 1.5s infinite linear;
}
@keyframes dot-typing {
    0% {
        box-shadow: 9984px 0 0 0 #9880ff, 9999px 0 0 0 #9880ff,
            10014px 0 0 0 #9880ff;
    }
    16.667% {
        box-shadow: 9984px -10px 0 0 #9880ff, 9999px 0 0 0 #9880ff,
            10014px 0 0 0 #9880ff;
    }
    33.333% {
        box-shadow: 9984px 0 0 0 #9880ff, 9999px 0 0 0 #9880ff,
            10014px 0 0 0 #9880ff;
    }
    50% {
        box-shadow: 9984px 0 0 0 #9880ff, 9999px -10px 0 0 #9880ff,
            10014px 0 0 0 #9880ff;
    }
    66.667% {
        box-shadow: 9984px 0 0 0 #9880ff, 9999px 0 0 0 #9880ff,
            10014px 0 0 0 #9880ff;
    }
    83.333% {
        box-shadow: 9984px 0 0 0 #9880ff, 9999px 0 0 0 #9880ff,
            10014px -10px 0 0 #9880ff;
    }
    100% {
        box-shadow: 9984px 0 0 0 #9880ff, 9999px 0 0 0 #9880ff,
            10014px 0 0 0 #9880ff;
    }
}
</style>
