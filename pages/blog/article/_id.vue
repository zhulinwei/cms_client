<template>
  <div class="l-detail-wrapper">
    <div class="l-detail">
      <p class="l-detail-title">{{ article.title }}</p>
      <p class="l-detail-subtitle">{{ article.author }}</p>
      <div class="l-detail-content" v-html="article.content"></div>
    </div>
    <l-comment :user="user" :comments="comments" @comment-success="commentSuccess"></l-comment>
  </div>
</template>

<script>
  import axios from 'axios';
  import moment from 'moment';

  import comment from './temp/comment.vue';

  const userRoleType = {
    USER: 'user',
    TOURIST: 'tourist'
  };

  export default {
    layout: 'blog',
    data() {
      let user = {};
      let article = {
        content: "<p style='text-align: center'>正在加载中...</p>"  
      };
      let comments = [];
      let articleId = '';
      let commentCount = 0;
      return { user, article, comments, articleId, commentCount };
    },
    created() {
      this.articleId = this.$route.params.id || '';
      this.query();
    },
    methods: {
       async query() {
        if (!this.articleId) return this.$notify.error('文章编号不存在');
        if (this.articleId.length != 24) return this.$notify.error('无效的文章编号');
        const [ user, article, comments ] = await Promise.all([
          axios.get('/api/users/info'),
          axios.get(`/api/blog/articles/${this.articleId}?inc=1`),
          axios.get(`/api/blog/articles/${this.articleId}/comments`),
        ]);
        this.article = article.data;
        this.user = user.data;
        this.comments = comments.data.list;
        this.commentCount = comments.data.count;
      },
      async commentSuccess() {
        const comments = await axios.get(`/api/blog/articles/${this.articleId}/comments`);
        this.comments = comments.data.list;
        this.commentCount = comments.data.count;
      }
    },
    components: {
      "l-comment": comment
    }
  }
</script>

<style>
  .l-detail-wrapper {
    /* line-height: 1.5; */
    /* font-size: 15px; */
    width: 100%; 
    max-width: 960px;
    margin: 20px auto 0;
    background-color: #fff;
  }
  .l-detail {
    padding: 0 10px;  
  }
  .l-detail-title {
    text-align: center;
    font-size: 20px; 
    padding: 5px 10px 2px;
  }
  .l-detail-subtitle {
    text-align: center;
  }
  .l-detail-content {
    font-size: 15px;
    line-height: 1.8em;  
  }
  /* table 样式 */
  .l-detail-wrapper table {
    border-top: 1px solid #ccc;
    border-left: 1px solid #ccc;
  }
  .l-detail-wrapper table td,
    table th {
    border-bottom: 1px solid #ccc;
    border-right: 1px solid #ccc;
    padding: 3px 5px;
  }
  .l-detail-wrapper table th {
    border-bottom: 2px solid #ccc;
    text-align: center;
  }
  /* blockquote 样式 */
  .l-detail-wrapper blockquote {
    display: block;
    border-left: 8px solid #d0e5f2;
    padding: 5px 10px;
    margin: 10px 0;
    line-height: 1.4;
    font-size: 100%;
    background-color: #f1f1f1;
  }

  /* code 样式 */
  .l-detail-wrapper code {
    display: inline-block;
    *display: inline;
    *zoom: 1;
    background-color: #f1f1f1;
    border-radius: 3px;
    padding: 3px 5px;
    margin: 0 3px;
    overflow: auto;
  }
  .l-detail-wrapper pre code {
    display: block;
  }

  /* ul ol 样式 */
  .l-detail-wrapper ul, 
  .l-detail-wrapper ol {
    margin: 10px 0 10px 20px;
  }
</style>
