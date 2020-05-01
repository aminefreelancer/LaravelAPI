<template>
    <div>
        <h2 class="mt-4 mb-2">List of articles</h2>
        <div v-for="article in articles" :key="article.id" class="card mb-2">
            <div class="card-body">
                <h5 class="card-title">{{article.title}}</h5>
                <p class="card-text">{{article.body}}</p>
            </div>
            <div class="card-footer">
                <small class="text-muted">Last updated 3 mins ago</small>
            </div>
        </div>
        <nav aria-label="Page navigation example">
            <ul class="pagination justify-content-center">
                <li v-bind:class="[{disabled : !pagination.prev_page_url}]" class="page-item">
                    <a class="page-link" href="#" aria-label="Previous" @click="fetchArticles(pagination.prev_page_url)">
                        <span aria-hidden="true">&laquo;</span>
                        <span class="sr-only">Previous</span>
                    </a>
                </li>
                <li class="page-item">
                    <a class="page-link text-dark" href="#">Page {{pagination.current_page}} of {{pagination.last_page}}</a>
                </li>
                <li v-bind:class="[{disabled : !pagination.next_page_url}]" class="page-item">
                    <a class="page-link" href="#" aria-label="Next" @click="fetchArticles(pagination.next_page_url)">
                        <span aria-hidden="true">&raquo;</span>
                        <span class="sr-only">Next</span>
                    </a>
                </li>
            </ul>
        </nav>

    </div>
</template>

<script>
    export default {
         data() {
            return {
                articles: [],
                article: {
                    id: '',
                    title: '',
                    body: ''
                },
                article_id: '',
                pagination: {},
                edit: false
            };
        },

        created() {
            this.fetchArticles();
        },

        methods: {
            fetchArticles(page_url) {
                let vm = this;
                page_url = page_url || '/api/articles';
                fetch(page_url)
                .then(res => res.json())
                .then(res => {
                    this.articles = res.data;
                    vm.makePagination(res.meta, res.links);
                })
                .catch(err => console.log(err));
            },
            makePagination(meta, links) {
                let pagination = {
                    current_page: meta.current_page,
                    last_page: meta.last_page,
                    next_page_url: links.next,
                    prev_page_url: links.prev
                };
                this.pagination = pagination;
            },
        }

    }
</script>

<style scoped>

</style>