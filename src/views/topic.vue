<template>
    <div>
        <nv-head page-type="主题"
                 :show-menu="showMenu"
                 :need-add="true"
                 :fix-head="true">
        </nv-head>
      <div id="page"
           :class="{'show-menu':showMenu}"
           v-if="topic.title">
        <h2 class="topic-title" v-text="topic.title"></h2>
        <section class="author-info">
          <img class="avatar" :src="topic.user.avatar" />
          <div class="col">
            <span>{{topic.user.username}}</span>
            <time>
              发布于:{{topic.inTime | getLastTimeStr(true)}}
            </time>
          </div>
          <div class="right">
                    <span class="tag"
                          :class="getTabInfo(topic.tab, topic.good, topic.top, true)"
                          v-text="getTabInfo(topic.tab, topic.good, topic.top, false)">
                    </span>
            <span class="name">{{topic.view}}次浏览</span>
          </div>
        </section>

        <section class='markdown-body topic-content' v-html="topic.content">

        </section>

        <h3 class="topic-reply">
          <strong>{{topic.replyCount}}</strong> 回复
        </h3>
        <section class="reply-list">
          <ul>
            <li v-for="item in replies">
              <section class="user">
                <img class="head" :src="item.user.avatar"/>
                <div class="info">
                  <span class="cl">
                    <span class="name" v-text="item.user.username"></span>
                    <span class="name mt10">
                      <span></span>
                      发布于:{{item.inTime | getLastTimeStr(true)}}
                    </span>
                  </span>
                   <span class="cr">
                                    <span class="iconfont icon "

                                          @click="upReply(item)">&#xe608;</span>
                                    <span class="iconfont icon" @click="addReply(item.id)">&#xe609;</span>
                    </span>

                </div>
              </section>
              <div class="reply_content" v-html="item.content"></div>

            </li>

          </ul>

        </section>
        <nv-top></nv-top>
        <nv-reply v-if="userInfo.id"
                  :topic="topic"
                  :replies="replies"
                  :topic-id="topicId">
        </nv-reply>

      </div>
      <div class='no-data' v-if="noData">
        <i class="iconfont icon-empty">&#xe60a;</i>
        该话题不存在!
      </div>
    </div>
</template>

<script>
  import $ from 'webpack-zepto';
  import utils from '../libs/utils.js';
  import nvHead from '../components/header.vue';
  import nvTop from '../components/backtotop.vue';
  import nvReply from '../components/reply.vue';
  import {
    mapGetters
  } from 'vuex';
  export default{
    data() {
      return {
          showMenu:true ,//是否展开菜单
          topic:{}, //主题
          replies:{}, //回复
          noData:false,
          topicId: '',
          curReplyId: ''
      }

    },
    computed: {
      ...mapGetters({
        userInfo: 'getUserInfo'
      })
    },
    mounted() {
      // 隐藏左侧展开菜单
      this.showMenu = false;
      // 获取url传的tab参数
      this.topicId = this.$route.params.id;

      // 加载主题数据
      $.get('http://localhost:8080/api/topic/' + this.topicId, (d) => {
        if (d && d.code==200 && d.detail.topic ) {
        this.topic = d.detail.topic;
        this.replies= d.detail.replies;
      } else {
        this.noData = true;
      }
      });
    },
    methods:{
      getTabInfo(tab, good = false, top, isClass) {
        return utils.getTabInfo(tab, good, top, isClass);
      },
      getLastTimeStr(time, ago) {
        return utils.getLastTimeStr(time, ago);
      },
      addReply(id) {
        this.curReplyId = id;

        if (!this.userInfo.id) {
          this.$router.push({
            name: 'login',
            params: {
              redirect: encodeURIComponent(this.$route.path)
            }
          });
        }
      },
      hideItemReply() {
        this.curReplyId = '';
      },
      upReply(item) {
        if (!this.userInfo.id) {
          this.$router.push({
            name: 'login',
            params: {
              redirect: encodeURIComponent(this.$route.path)
            }
          });
        } else {
          $.ajax({
              type: 'POST',
              url: 'https://cnodejs.org/api/v1/reply/' + item.id + '/ups',
              data: {
                accesstoken: this.userInfo.token
              },
              dataType: 'json',
              success: (res) => {
              if (res.success) {
            if (res.action === 'down') {
              let index = $.inArray(this.userInfo.userId, item.ups);
              item.ups.splice(index, 1);
            } else {
              item.ups.push(this.userInfo.userId);
            }
          }
        },
          error: (res) => {
            let error = JSON.parse(res.responseText);
            this.$alert(error.error_msg);
            return false;
          }
        });
        }
      }
    },
    components:{
        nvHead,
        nvTop,
        nvReply
    }
  }

</script>
