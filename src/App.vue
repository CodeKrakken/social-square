
<script setup lang="ts">
import { ref } from 'vue'

interface Post {
  id: number
  author: string
  content: string
  likes: number
  liked: boolean
}

const newPost = ref('')

const posts = ref<Post[]>([
  {
    id: 1,
    author: 'Alex Morgan',
    content: 'Just finished a weekend hike. Getting outside makes such a difference!',
    likes: 4,
    liked: false,
  },
  {
    id: 2,
    author: 'Sam Taylor',
    content: 'What is everyone learning at the moment?',
    likes: 2,
    liked: false,
  },
])

async function loadPosts() {
  const response = await fetch('http://localhost:3000/posts')

  if (!response.ok) {
    throw new Error('Failed to load posts')
  }

  posts.value = await response.json()
}

function publishPost() {
  const content = newPost.value.trim()

  if (!content) return

  posts.value.unshift({
    id: Date.now(),
    author: 'You',
    content,
    likes: 0,
    liked: false,
  })

  newPost.value = ''
}

function toggleLike(post: Post) {
  post.liked = !post.liked
  post.likes += post.liked ? 1 : -1
}
</script>

<template>
  <main class="app">
    <header>
      <h1>SocialSquare</h1>
      <p>A little corner of the internet.</p>
    </header>

    <section class="composer">
      <form @submit.prevent="publishPost">
        <label for="new-post">What's going down, Charlie Brown?</label>
        <textarea
          id="new-post"
          v-model="newPost"
          placeholder="Write a post..."
          rows="3"
          maxlength="500"
        />
        <button type="submit" :disabled="!newPost.trim()">
          Publish
        </button>
      </form>
    </section>

    <section class="feed" aria-label="Recent posts">
      <h2 v-if="posts.length !== 0">Recent posts</h2>
      <article v-for="post in posts" :key="post.id" class="post">
        <h3>{{ post.author }}</h3>
        <p>{{ post.content }}</p>

        <button
          type="button"
          :aria-pressed="post.liked"
          @click="toggleLike(post)"
        >
          {{ post.liked ? 'Unlike' : 'Like' }} ({{ post.likes }})
        </button>
      </article>

      <p v-if="posts.length === 0">No posts yet.</p>
    </section>
  </main>
</template>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  background: #f3f5f8;
  color: #202938;
  font-family: Arial, sans-serif;
}

button,
textarea {
  font: inherit;
}

.app {
  width: min(100% - 32px, 640px);
  margin: 0 auto;
  padding: 32px 0;
}

header {
  margin-bottom: 24px;
}

header h1 {
  margin-bottom: 4px;
}

header p {
  color: #657184;
}

.composer,
.post {
  padding: 20px;
  margin-bottom: 16px;
  background: white;
  border: 1px solid #e0e5ec;
  border-radius: 12px;
}

form {
  display: grid;
  gap: 12px;
}

textarea {
  width: 100%;
  padding: 12px;
  border: 1px solid #cbd2dc;
  border-radius: 8px;
  resize: vertical;
}

button {
  width: fit-content;
  padding: 9px 16px;
  border: 0;
  border-radius: 6px;
  background: #2459c4;
  color: white;
  cursor: pointer;
}

button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.post h3 {
  margin-top: 0;
}

.post p {
  line-height: 1.5;
  overflow-wrap: anywhere;
}

.post button {
  background: #edf2fc;
  color: #2459c4;
}

.post button[aria-pressed="true"] {
  background: #2459c4;
  color: white;
}

.feed h2 {
  font-size: 1.2rem;
}
</style>
