<template>
  <div v-if="comments.length" class="comments-container">
    <ul>
      <li v-for="comment in comments" :key="comment.id" class="comment">
        <p><strong>{{ comment.by }}</strong> - {{ timeAgo(comment.time) }} ago</p>
        <p v-html="decodeHtmlEntities(comment.text)"></p>
      </li>
    </ul>
  </div>
</template>

<script>
import he from 'he';

export default {
  name: 'CommentSection',
  props: {
    postId: {
      type: Number,
      required: true,
    },
  },
  data() {
    return {
      comments: [],
    };
  },
  async mounted() {
    await this.fetchComments();
  },
  methods: {
    async fetchComments() {
      try {
        const res = await fetch(`https://hacker-news.firebaseio.com/v0/item/${this.postId}.json`);
        const post = await res.json();
        if (post && post.kids) {
          const commentPromises = post.kids.slice(0, 5).map(id =>
              fetch(`https://hacker-news.firebaseio.com/v0/item/${id}.json`).then(response => response.json())
          );
          this.comments = await Promise.all(commentPromises);
        }
      } catch (error) {
        console.error('Error fetching comments:', error);
      }
    },
    timeAgo(timestamp) {
      const now = Date.now() / 1000;
      const secondsAgo = now - timestamp;
      if (secondsAgo < 3600) return Math.floor(secondsAgo / 60) + ' minutes';
      else if (secondsAgo < 86400) return Math.floor(secondsAgo / 3600) + ' hours';
      return Math.floor(secondsAgo / 86400) + ' days';
    },
    decodeHtmlEntities(text) {
      return he.decode(text);
    },
  },
};
</script>

<style scoped>
.comments-container {
  max-height: 200px;
  overflow-y: auto;
  border-top: 1px solid #ddd;
  padding-top: 10px;
}

.comment ul {
  padding-left: 0;
  margin: 0;
}

.comment {
  margin-bottom: 10px;
  list-style-type: none;
}

.comment p {
  margin: 0;
  font-size: 14px;
  text-align: left;
}
</style>
