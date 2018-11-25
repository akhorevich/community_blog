<template>
    <div class="container">
        <div class="row">
            <div class="col-md-8 offset-md-2">
                <div class="card" v-if="!loading">
                    <img height="420px" :src="article.imageUrl" alt="" class="card-img-top">
                    <div class="card-body">
                        <div class="card-title text-center my-3">
                            <h1>{{ article.title }}</h1>
                        </div>
                        <div class="article-content" v-html="article.content"></div>
                        <div class="comments my-4">
                                <vue-disqus shortname="Nordpage" :identifier="article.slug" :url="url"></vue-disqus>
                        </div>
                    </div>
                </div>
                <div class="loader text-center my-3" v-else>
                    <i class="fas fa-5x fa-spin fa-spinner"></i>
                </div>
            </div>
        </div>
    </div>
</template>

<script>

import Axios from 'axios';
import config from '@/config';

export default {
    mounted(){
        this.getArticle();
    },
    data(){
        return{
            article: {},
            loading: true,
            url: window.location.href
        }
    },
    methods: {
        getArticle(){
            Axios.get(`${config.apiUrl}/article/${this.$route.params.id}`).then(response => {
                this.loading = false;
                this.article = response.data.data;
            });
        }
    }
}
</script>
