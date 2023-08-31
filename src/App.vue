<script setup>
import axios from 'axios';
import { reactive, onMounted } from 'vue'
// import {useToast} from 'vue-toast-notification'

import { toast } from 'vue3-toastify';
import 'vue3-toastify/dist/index.css';

axios.defaults.baseURL = "https://basic-blog.teamrabbil.com/api"

// const toast = useToast()

const appData = reactive({
    appname: "Ostad Blog",
    catagories: [],
    curCatagory: 0,
    curPosition: 0,
    post: {
    },
    posts: [],
    page_mode: "post" // "list"
})

const fetch_blog_detail = async () => {
    const resp = await axios.get("/post-details/" + appData.curPosition)
    appData.post = resp.data['postDetails']
    if (resp.data["postDetails"] == null){
        // alert("no data found")
        toast.error('This post does not have any more details!!',{autoClose: 1500,});

    } else {
        appData.page_mode = "post"
    }

}
const page_init = async () => {

    let resp = await axios.get("/post-categories")
    appData.catagories = resp.data

    resp = await axios.get("/post-newest")
    // appData.curCatagory = resp.data[0]['category_id']
    appData.curCatagory = 0
    appData.curPosition = resp.data[0]['id']

    await fetch_blog_detail()

}
onMounted(async () => {

    await page_init()

})

const list_posts = async (id) => {
    // console.log(id)
    const resp = await axios.get("/post-list/" + id)
    appData.posts = resp.data
    appData.page_mode = 'list'

    appData.curCatagory = id
}

const show_post= async (id)=>{
    // console.log(id)
    appData.curPosition = id
    await fetch_blog_detail()
}

</script>


<template>
    <div class="h-screen">
        <div class="flex justify-between items-center p-3 border-2">
            <span class="text-sm font-bold px-2 py-1">
                <a href="/" @click.prevent="page_init()">{{ appData.appname }}</a>
            </span>
            <!-- {{ appData.curCatagory }} -->
            <nav class="mx-10">
                <ul class="flex space-x-5 justify-center items-center">
                    <li class="cursor-pointer px-2 shadow-gray-400 shadow-lg hover:bg-gray-800 hover:text-white hover:text-xl hover:rounded-md"
                        @click.prevent="page_init()"
                        :class="appData.curCatagory == 0 ? 'bg-slate-900 text-white' : ''">
                        হোম
                    </li>
                    <li @click.prevent="list_posts(catagory.id)"
                        class="cursor-pointer hover:bg-gray-800 hover:text-white hover:text-xl hover:rounded-md"
                        :class="appData.curCatagory == catagory.id ? 'bg-slate-900 text-white px-2' : ''"
                        v-for="catagory, index in appData.catagories" :key="index">
                        {{ catagory.name }}
                    </li>
                </ul>
            </nav>
        </div>
        <template v-if="appData.page_mode == 'list'">
            <div class="grid grid-cols-2 mt-5 md:grid-cols-3 md:m-20">

                <div 
                    v-for="post, index in appData.posts" 
                    :key="index" 
                    class="flex flex-col justify-center items-center border-2 m-5"
                >
                <!-- {{ post }} -->
                    <img class="rounded-lg" :src="post.img" :alt="post.title">
                    <div class="m-5 flex flex-col justify-center items-start">
                        <span class="text-sm font-bold mt-2">{{ post.title }}</span>
                        <p class="mt-2 text-sm">
                            {{ post.short }}
                        </p>
                    </div>
                    <div class="flex justify-between items-center w-full pl-5 mb-4">
                        <span>ID: {{ post.id }}</span>
                        <button 
                            class="cursor-pointer bg-blue-700 text-white border-2 rounded-lg px-2 py-1 mr-5"
                            @click.prevent="show_post(post.id)"
                        >
                            more
                        </button>
                        </div>
                </div>
            </div>
        </template>
        <div v-if="appData.page_mode == 'post'" class="flex flex-col justify-center items-center mt-5">
            <!-- {{ appData.post }} -->
            <img class="rounded-lg max-w-sm  md:max-w-5xl" :src="appData.post.img" :alt="appData.post.title">
            <div class="flex flex-col justify-center items-start  max-w-sm  md:max-w-5xl border-2 p-5">
                <span class="text-lg font-bold">{{ appData.post.title }}</span>
                <p class="mt-2 text-sm">
                    {{ appData.post.content }}
                </p>
            </div>
        </div>
    </div>
</template>

<style scoped></style>
