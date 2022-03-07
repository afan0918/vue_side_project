<template>
    <div>
        <h2 v-if="firstView">輸入想查詢的用戶名稱</h2>
        <h2 v-if="loading">Loading...</h2>
        <h2 v-if="errorMsg">{{errorMsg}}</h2>
        <div class="row">
            <div class="card" v-for="(user, index) in users" :key="index">
                <a :href="user.url" target="_blank">
                <img :src="user.avatar_url" style='width: 100px'/>
                </a>
                <p class="card-text">{{user.name}}</p>
            </div>
        </div>
    </div>
</template>

<script>
    import PubSub from 'pubsub-js'
    import axios from 'axios'
    export default {
        data () {
            return {
                firstView:true,
                loading: false,
                // [{url:'', avatar_url:'', name:''}]
                users: null,
                errorMsg: '',
            }
        },
        mounted () {
            //訂閱搜索的消息
            PubSub.subscribe('search', (msg, searchName) =>{
                //發送ajxa請求進行搜索
                const url = `https://api.github.com/search/users?q=${searchName}`;
                
                //更新狀態
                this.loading = true;
                this.firstView = false;
                this.errorMsg = null;
                this.users = null;

                //發送ajax請求
                axios.get(url).then(response =>{
                    const result = response.data;
                    const users = result.items.map(item => ({
                        url:item.html_url, 
                        avatar_url: item.avatar_url, 
                        name:item.login
                    }));
                    //成功
                    this.loading = false;
                    this.users = users;
                }).catch(error =>{
                    //失敗
                    this.loading = false
                    this.errorMsg = '請求失敗'
                })
            })
        }
    }
</script>

<style>
    .card {
        float: left;
        width: 33.333%;
        padding: .75rem;
        margin-bottom: 2rem;
        border: 1px solid #efefef;
        text-align: center;
    }

    .card > img {
        margin-bottom: .75rem;
        border-radius: 100px;
    }

    .card-text {
        font-size: 85%;
    }
</style>
