<template>
  <Header />
  <main class="container">
    <!-- <feed-preview url="http://photojournal.jpl.nasa.gov/rss/new" /> -->
    <feed-preview url="https://www.gamespot.com/feeds/mashup/" />
  </main>
</template>

<script>
import Header from "./components/Header.vue";
import FeedPreview from "./components/FeedPreview.vue";

export default {
  components: { FeedPreview, Header },
};
</script>

