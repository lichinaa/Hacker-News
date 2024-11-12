<template>
  <div class="page-container">
    <!-- Header section -->
    <div class="header">
      <div class="logo-container">
        <img src="@/assets/image/HN_logo.jpg" alt="Hacker News Logo" class="logo" />
        <span class="header-title">Search Hacker News</span>
      </div>
      <div class="search-bar-container">
        <input
            v-model="searchQuery"
            type="text"
            placeholder="Search by title"
            class="search-bar"
        />
      </div>
    </div>

    <!-- Post list section -->
    <div class="post-list">
      <h1>Latest Posts</h1>
      <ul>
        <li v-for="post in filteredPosts" :key="post.id" class="post-card">
          <img src="https://via.placeholder.com/100" alt="Post Image" class="post-image" />
          <div class="post-content">
            <a :href="post.url" target="_blank" rel="noopener noreferrer" class="post-title">
              {{ post.title }}
            </a>
            <p class="post-details">
              <span>{{ post.score || 0 }} points</span>
              <span>{{ post.by }}</span>
              <span>{{ timeAgo(post.time) }} ago</span>
              <a :href="`https://news.ycombinator.com/item?id=${post.id}`" target="_blank" class="post-link">
                www.hackernews.com
              </a>
              <span class="comment-count" @click="toggleComments(post.id)">
                {{ post.kids?.length || 0 }} comments
              </span>
            </p>
            <!-- Show CommentSection -->
            <CommentSection v-if="showComments[post.id]" :postId="post.id" :decodeHtmlEntities="decodeHtmlEntities"/>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>


<script>
import CommentSection from "./CommentSection.vue";
import he from 'he';
import '@/assets/css/post-list.css'

export default {
  name: 'PostList',
  components: {
    CommentSection,
  },

  data() {
    return {
      posts: [],
      showComments: {},
      searchQuery: '',
    };
  },
  computed: {
    filteredPosts() {
      return this.posts.filter(post =>
          post.title.toLowerCase().includes(this.searchQuery.toLowerCase())
      );
    },
  },
  mounted() {
    this.fetchPosts();
  },
  methods: {
    async fetchPosts() {
      try {
        const cachedPosts = await caches.match('https://hacker-news.firebaseio.com/v0/topstories.json');

        if (cachedPosts) {
          const postIds = await cachedPosts.json();
          const postPromises = postIds.slice(0, 10).map(id =>
              caches.match(`https://hacker-news.firebaseio.com/v0/item/${id}.json`).then(response => response ? response.json() : null)
          );
          this.posts = await Promise.all(postPromises);
        } else {
          const res = await fetch('https://hacker-news.firebaseio.com/v0/topstories.json');
          const postIds = await res.json();
          const postPromises = postIds.slice(0, 10).map(id =>
              fetch(`https://hacker-news.firebaseio.com/v0/item/${id}.json`).then(response => response.json())
          );
          this.posts = await Promise.all(postPromises);
        }
      } catch (error) {
        console.error('Error fetching posts:', error);
      }
    },
    timeAgo(timestamp) {
      const now = Date.now() / 1000;
      const secondsAgo = now - timestamp;

      if (secondsAgo < 3600) {
        return Math.floor(secondsAgo / 60) + ' minutes';
      } else if (secondsAgo < 86400) {
        return Math.floor(secondsAgo / 3600) + ' hours';
      } else {
        return Math.floor(secondsAgo / 86400) + ' days';
      }
    },
    toggleComments(postId) {
      this.showComments[postId] = !this.showComments[postId];
    },
    decodeHtmlEntities(text) {
      return he.decode(text);
    },
  },
};
</script>
