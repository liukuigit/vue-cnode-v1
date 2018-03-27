<template>
  <div>
    <nv-head page-type="用户信息" :fix-head="true" :show-menu="false" :need-add="true" ></nv-head>
    <section class="userinfo">
      <img class="u-img" :src="user.avatar" /><br/>
      <span class="u-name" v-text="user.username"></span>
      <div class="u-bottom">
        <span class="u-time" v-text="getLastTimeStr(user.inTime, false)"></span>
        <span class="u-score">积分：</span>
      </div>
    </section>
    <section class="topics">
      <ul class="user-tabs">
        <li class="item br" :class='{"selected":selectItem === 1}' @click="changeItem(1)">最近回复</li>
        <li class="item" :class='{"selected":selectItem === 2}' @click="changeItem(2)">最新发布</li>
      </ul>
      <div class="message" v-for="item in currentData">
        <section class="user">
          <router-link class="head" :to="{name:'user',params:{username:item.user.username}}">
            <img  :src="item.user.avatar" />
          </router-link>
          <router-link class="info" :to="{name:'topic',params:{id:item.id}}">
            <div class="t-title" v-if="item.title" >{{item.title}}</div>
            <div class="t-title" v-if="item.content" >{{item.content}}</div>
                        <span class="cl mt12">
                            <span class="name">{{item.user.username}}</span>
                        </span>
                        <span class="cr mt12">
                            <span class="name" v-text="getLastTimeStr(item.inTime, true)"></span>
                        </span>
          </router-link>
        </section>
      </div>
      <div class="no-data" v-show="currentData.length === 0">
        <i class="iconfont icon-empty">&#xe60a;</i>
        暂无数据!
      </div>
    </section>
  </div>
</template>
<script>
  import $ from 'webpack-zepto';
  import utils from '../libs/utils.js';
  import nvHead from '../components/header.vue';

  export default {
    data() {
      return {
        user: {},
        currentData: [],
        selectItem: 1,
        topics:[],
        replys:[]
      }
    },
    mounted() {
      this.getUser();
    },
    methods: {
      // 切换tab
      changeItem(idx) {
        this.selectItem = idx;
        this.currentData = idx === 1 ? this.replys : this.topics;
      },
      getLastTimeStr(date, friendly) {
        return utils.getLastTimeStr(date, friendly);
      },
      getUser() {
        let username = this.$route.params.loginname;

        if (!username) {
          this.$alert('缺少用户名参数');
          this.$router.push({
            path: '/'
          });
          return false;
        }
        $.get('http://localhost:8080/api/user/' + username, (d) => {
          if (d && d.detail) {
          let data = d.detail;
          this.user = data.user;

          if (data.replyPage.content.length > 0) {
            this.currentData = data.replyPage.content;
            this.replys=data.replyPage.content;
          }
          if (data.topicPage.content.length > 0) {
            this.currentData = data.topicPage.content;
            this.selectItem = 2;
            this.topics=data.topicPage.content;
            }
        }
      });
      }
    },
    watch: {
      // 切换页面
      '$route' (to, from) {
        this.getUser();
      }
    },
    components: {
      nvHead
    }
  };
</script>
