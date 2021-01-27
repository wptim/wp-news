<template>
  <h2 v-if="loading > 0">
    Loading...
  </h2>
  <div v-else>
    <article>
      <h1>{{ post.title }}</h1>
      <div class="placeholder">
        <img :alt="post.title" :src="post.featuredImage.node.sourceUrl" />
      </div>
      <vue-markdown>{{ post.content }}</vue-markdown>
    </article>
  </div>
</template>

<script>
import gql from "graphql-tag";
import VueMarkdown from "vue-markdown";

const post = gql`
  query post($id: ID!) {
    post(id: $id) {
      id
      title
      slug
      featuredImage {
        node {
          sourceUrl(size: LARGE)
        }
      }
      content
    }
  }
`;

export default {
  name: "Post",
  data: () => ({
    loading: 0,
  }),
  apollo: {
    $loadingKey: "loading",
    post: {
      query: post,
      variables() {
        return {
          id: this.$route.params.slug,
        };
      },
    },
  },
  components: { VueMarkdown },
};
</script>

<style scoped>
.placeholder {
  height: 600px;
  background-color: #eee;
}
</style>
