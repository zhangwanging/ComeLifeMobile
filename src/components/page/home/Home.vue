<template>
    <div class="container-y">
        <div v-if="isNormal">
            <div class="container-y" v-for="(item, index) in lists" :key="item._id">
                <div class="container-x app-content-padding">
                    <img :src="item.avatarUrl" alt="" class="avatar">
                    <div class="container-y container-item">
                        <div class="item-name">{{item.nickName}}</div>
                        <div class="item-content">{{item.jokeContent}}</div>
                        <div class="item-footer">
                            <span>{{item.publishTime}}</span>
                            <div v-if="isLogin">
                                <img class="item-favorite" v-if="item.isFavorite" src="./favorite_selected.png" alt=""
                                     @click="favoriteDel(item,index)">
                                <img class="item-favorite" v-else src="./favorite_unselected.png" alt=""
                                     @click="favoriteAdd(item,index)">
                            </div>
                        </div>
                    </div>
                </div>
                <div class="item-line"/>
            </div>
        </div>
        <div class="error" v-else>
            <img src="../../../assets/img/error.png" alt="">
            <div class="error-text">{{errorMsg}}</div>
        </div>
    </div>
</template>

<script>
    //TODO:得到isLogin的状态后，更新isFavorite,不然点击收藏会报错

    export default {
        name: "Home",
        data() {
            return {
                isNormal:true,
                errorMsg:'',
                isLogin: false,
                lists: []
            }
        },
        created() {
            this.fetchData()
            if (!this.global.isLogin) {
                this.autoLogin()
            } else {
                this.isLogin = true;
            }
        },
        methods: {
            fetchData() {
                var _this = this;
                this.request.post('/allUserJoke')
                    .then(function (response) {
                        var r = response.data;
                        if(r.code===0){
                            if(r.data.length!==0){
                                var d = _this.dataHandler(r);
                                _this.lists = d;
                                _this.isNormal=true;
                            }else{
                                _this.isNormal=false;
                                _this.errorMsg='空空如也~'
                            }

                        }else{
                            _this.isNormal=false;
                            _this.errorMsg='数据加载出错~'
                        }

                    })
                    .catch(function (error) {
                        console.log(error);
                        _this.isNormal=false;
                        _this.errorMsg='数据加载出错~'
                    });
            },
            dataHandler: function (data) { //响应的对象
                var _this = this;
                var users = data.data;
                var arr = [];
                for (var i = 0; i < users.length; i++) {
                    var user = users[i];
                    var r = user.jokes;
                    for (var j = 0; j < r.length; j++) {
                        var joke = r[j];
                        joke.uid = user._id;
                        joke.nickName = user.nickName;
                        joke.gender = user.gender;
                        joke.city = user.city;
                        joke.province = user.province;
                        joke.country = user.country;
                        joke.avatarUrl = user.avatarUrl;
                        joke.isFavorite = false;
                        var collector = joke.collectors;
                        var userMsg = _this.util.storage.getObj(_this.util.storage.key.LOGIN);
                        var user_id = userMsg && userMsg.user_id;
                        for (var k = 0; k < collector.length; k++) {
                            if (user_id === collector[k]) {
                                joke.isFavorite = true;
                                break;
                            }
                        }
                        arr.push(joke);
                    }
                }
                return arr;
            },
            favoriteAdd(item, index) {
                var joke_id = item._id;
                var s = this.util.storage;
                var user_id = s.getObj(s.key.LOGIN).user_id;
                var _this = this;
                this.request.post('/jokeCollectorAdd', {
                    joke_id: joke_id,
                    user_id: user_id
                })
                    .then(function (res) {
                        var r = res.data;
                        if (r.code === 0) {
                            _this.lists[index].isFavorite = true;
                        } else {
                            _this.$toast('添加失败~')
                        }
                    })
                    .catch(function (err) {
                        _this.$toast('添加失败~')
                        console.log(err);
                    })
            },
            favoriteDel(item, index) {
                var joke_id = item._id;
                var s = this.util.storage;
                var user_id = s.getObj(s.key.LOGIN).user_id;
                var _this = this;
                this.request.post('/jokeCollectorRemove', {
                    joke_id: joke_id,
                    user_id: user_id
                })
                    .then(function (res) {
                        var r = res.data;
                        if (r.code === 0) {
                            _this.lists[index].isFavorite = false;
                        } else {
                            _this.$toast('移除失败~')
                        }
                    })
                    .catch(function (err) {
                        console.log(err)
                        _this.$toast('移除失败~')
                    })
            },
            autoLogin() {
                var _this = this;
                var s = this.util.storage
                var loginMsg = s.getObj(s.key.LOGIN)
                console.log('loginMsg...:' + JSON.stringify(loginMsg))
                if (loginMsg) {
                    this.request.post('/login', loginMsg)
                        .then(function (res) {
                            var r = res.data;
                            if (r.code === 0) {
                                _this.global.isLogin = true;
                                _this.global.userData = r.data[0];
                                _this.isLogin = true;
                                console.log('autoLogin success')
                            } else {
                                console.log('autoLogin failed:msg error');
                            }
                        })
                        .catch(function (err) {
                            console.log(err)
                        })
                } else {
                    console.log('autoLogin failed:no login msg');
                }
            }
        }
    }
</script>

<style scoped>

    .container-x {
        width: 100%;
        display: flex;
        justify-content: space-between;
        box-sizing: border-box;
        background-color: white;
    }

    .avatar {
        width: 36px;
        height: 36px;
        border-radius: 18px;
    }

    .container-item {
        padding-left: 10px;
        justify-content: space-between;
        box-sizing: border-box;
    }

    .item-name {
        height: 36px;
        line-height: 36px;
    }

    .item-content {
        min-height: 50px;
        display: flex;
        align-items: center;
    }

    .item-footer {
        display: flex;
        justify-content: space-between;
    }

    .item-line {
        width: 100%;
        height: 1px;
    }

    .item-favorite {
        width: 20px;
        height: 20px;
    }
    /*err*/
    .error{
        width: 100%;
        position: absolute;
        top:25%;
        text-align: center;
    }
    .error-text{
        margin-top:10px;
    }

</style>
