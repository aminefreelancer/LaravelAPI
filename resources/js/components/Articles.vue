<template>
    <div id="articles" class="row">
        <div class="col position-sticky">
            <h2 v-if="edit" class="mt-4 mb-2">
                Edit Article
            </h2>
            <h2 v-else class="mt-4 mb-2">
                New Article
            </h2>
            <div class="card">
                <form @submit.prevent="addArticle">
                    <div class="card-body">
                        <div class="form-group">
                            <label for="title" class="col-form-label">Title:</label>
                            <input type="text" class="form-control" required id="title" v-model="article.title">
                        </div>
                        <div class="form-group">
                            <label for="message-text" class="col-form-label">Body:</label>
                            <textarea class="form-control" id="message-text" required rows="5" v-model="article.body"></textarea>
                        </div>
                        <button type="submit" class="btn btn-primary">Save</button>
                    </div>
                </form>
            </div>
        </div>
        <div class="col-8">
            <h2 class="mt-4 mb-2">List of articles</h2>
            <div v-if="action" class="alert alert-success fade show" role="alert">
                {{message}}
            </div>
            <div v-for="article in articles" :key="article.id" class="card mb-2">
                <div class="card-body">
                    <h5 class="card-title">
                        {{article.title}}
                    </h5>
                    <p class="card-text">{{article.body}}</p>
                </div>
                <div class="card-footer">
                    <small class="text-muted">Last updated {{dateShow(article.updated)}}</small>
                    <button @click="deleteArticle(article.id)" type="button" class="btn btn-danger btn-sm float-right ml-2">Delete</button>
                    <button @click="editArticle(article)" 
                            data-toggle="modal" 
                            data-target="#articleModal" 
                            type="button" 
                            class="btn btn-success btn-sm float-right">
                            Edit
                    </button>
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
    </div>
</template>

<script>
    import moment from 'moment';
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
                message: '',
                action: false,
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
            addArticle(){
                if(!this.edit) {
                    //Add new article
                    fetch('/api/article', {
                        method: 'post',
                        body: JSON.stringify(this.article),
                        headers: {
                            'content-type': 'application/json'
                        }
                    })
                    .then(res => res.json())
                    .then(data => {
                        this.clearForm();
                        this.fetchArticles();
                        this.message = "New article added with success !";
                        this.action= true;
                        //Display Flash Message for 3s
                        setTimeout(this.disableAlert, 3000);
                    })
                    .catch(err => console.log(err));
                    
                } else {
                    //Update existing article
                    fetch('/api/article', {
                        method: 'put',
                        body: JSON.stringify(this.article),
                        headers: {
                            'content-type': 'application/json'
                        }
                    })
                    .then(res => res.json())
                    .then(data => {
                        this.clearForm();
                        this.fetchArticles();
                        this.message = "Article updated with success !";
                        this.action= true;
                        //Display Flash Message for 3s
                        setTimeout(this.disableAlert, 3000);
                    })
                    .catch(err => console.log(err));
                }
            },
            editArticle(article) {
                this.edit= true;
                this.article.article_id = article.id;
                this.article.id = article.id;
                this.article.title = article.title;
                this.article.body = article.body;
            },
            deleteArticle(id) {
                if(confirm('Are you sure ?')) {
                    fetch(`/api/article/${id}`, {
                        method: 'delete'
                    })
                    .then(res => res.json())
                    .then(res => {
                        this.fetchArticles();
                        this.message = "Article deleted !";
                        this.action= true;
                        setTimeout(this.disableAlert, 3000);
                    })
                    .catch(err => console.log(err));
                }
            },
            disableAlert() {
                this.message = '';
                this.action = false;
            },
            clearForm() {
                this.edit = false;
                this.article.id = null;
                this.article.article_id = null;
                this.article.title = '';
                this.article.body = '';
            },
            dateShow(date) {
                return moment(date).fromNow();
            }
        }

    }
</script>

<style scoped>
#articles {
    margin:0 15px;
}
</style>