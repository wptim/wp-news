<template>
  <div>
    <section v-if="posts">
      <ul>
        <li v-for="post in posts.nodes" :key="post.id">
          <router-link :to="`/post/${post.id}`" class="link">
            <div class="placeholder">
              <img :alt="post.title" :src="post.featuredImage.node.sourceUrl" />
            </div>
            <h3>{{ post.title }}</h3>
          </router-link>
        </li>
      </ul>
      <button v-if="showMoreEnabled" @click="loadMorePosts">
        {{ loading ? "Loading..." : "Show more" }}
      </button>
    </section>
    <h2 v-else>
      Loading...
    </h2>
  </div>
</template>

<script>
import gql from "graphql-tag";

const POSTS_PER_PAGE = 10;

const posts = gql`
  query posts($first: Int!, $after: String) {
    posts(first: $first, after: $after) {
      pageInfo {
        hasNextPage
        endCursor
      }
      nodes {
        id
        slug
        title
        featuredImage {
          node {
            sourceUrl(size: THUMBNAIL)
          }
        }
      }
    }
  }
`;

export default {
  name: "Blog",
  data: () => ({
    loading: 0,
    showMoreEnabled: true,
  }),
  apollo: {
    $loadingKey: "loading",
    posts: {
      query: posts,
      variables: {
        first: POSTS_PER_PAGE,
      },
    },
  },
  methods: {
    loadMorePosts() {
      this.$apollo.queries.posts.fetchMore({
        variables: {
          after: this.posts.pageInfo.endCursor,
        },
        updateQuery: (previousResult, { fetchMoreResult }) => {
          const newPosts = fetchMoreResult.posts;
          this.showMoreEnabled = newPosts.pageInfo.hasNextPage;

          return {
            posts: {
              __typename: previousResult.posts.__typename,
              // Merging the tag list
              nodes: [...previousResult.posts.nodes, ...newPosts.nodes],
              pageInfo: newPosts.pageInfo,
            },
          };
        },
      });
    },
  },
};
</script>

<style scoped>
ul {
  padding: 0;
}
li {
  display: flex;
  align-items: center;
  margin-bottom: 16px;
  border: 1px solid #eee;
  overflow: hidden;
  border-radius: 5px;
}
.link {
  display: flex;
  color: #000;
}
.link:hover {
  box-shadow: 1px 1px 5px #999;
}
.placeholder {
  background-color: #eee;
  min-width: 150px;
  margin-right: 24px;
}
img {
  display: block;
  height: 100%;
}
.show-more-wrapper {
  display: flex;
  justify-content: center;
}
button {
  width: 100%;
  font-size: 16px;
  color: white;
  text-transform: uppercase;
  font-weight: bold;
  padding: 16px 24px;
  background: deepskyblue;
  border: none;
  border-radius: 0;
  cursor: pointer;
}
</style>
