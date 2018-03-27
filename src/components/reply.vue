<template>

    <section class="reply">
        <textarea id="content" rows="8" class="text"
            :class="{'err':hasErr}"
            v-model="content"
            placeholder='请输入回复内容...'>
        </textarea>
        <a class="button" @click="addReply">确定</a>
    </section>

</template>
<script>
    import $ from 'webpack-zepto';
    const utils = require('../libs/utils');
    const markdown = require('markdown').markdown;
    import {
      mapGetters
    } from 'vuex';

    export default {
        replace: true,
        props: ['topic', 'replyId', 'topicId', 'replyTo', 'show','replies'],
        data() {
            return {
                hasErr: false,
                content: '',
                author_txt: '<br/><br/><a class="form" href="https://github.com/shinygang/Vue-cnodejs">I‘m webapp-cnodejs-vue</a>'
            };
        },
        computed: {
          ...mapGetters({
            userInfo: 'getUserInfo'
          })
        },
        mounted() {
            if (this.replyTo) {
                this.content = `@${this.replyTo} `;
            }
        },
        methods: {
            addReply() {
                if (!this.content) {
                    this.hasErr = true;
                } else {
                    let time = new Date();
                    let linkUsers = utils.linkUsers(this.content);
                    //let htmlText = markdown.toHTML(linkUsers) + this.author_txt;
                    let htmlText = markdown.toHTML(linkUsers);
                    let replyContent = $('<div class="markdown-text"></div>').append(htmlText)[0].outerHTML;
                    let postData = {
                        //token: this.userInfo.token,
                        token:'4db3b62c-f595-43bf-a08d-82897a669cb7',
                        content: this.content
                    };
                    postData.topicId=this.topicId;
                    if (this.replyId) {
                        postData.reply_id = this.replyId;
                    }
                    $.ajax({
                        type: 'POST',
                        url: 'http://localhost:8080/api/reply/save',
                        data: postData,
                        dataType: 'json',
                        success: (res) => {
                            if (res.code ==200) {
                                this.topic.replyCount+=1;
                                this.replies.push({
                                    id: res.code,
                                    user: {
//                                        loginname: this.userInfo.loginname,
//                                        avatar_url: this.userInfo.avatar_url
                                      username: 'aaa',
                                      avatar: 'http://localhost:8080/static/images/upload/avatar/default.png'
                                    },
                                    content: replyContent,
                                    inTime: time
                                });
                            }
                            this.content = '';
                            if (this.show) {
                                this.$emit('close');
                            }
                        },
                        error: (res) => {
                            var error = JSON.parse(res.responseText);
                            this.$alert(error.error_msg);
                            return false;
                        }
                    });
                }
            }
        }
    };
</script>
