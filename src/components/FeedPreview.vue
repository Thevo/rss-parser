<template>

  <div v-if="!loading && !error" class="feed-container">
    <div class="channel">
      <div class="channel__row">
        <p class="channel__row-badge">Tittle:</p>

        <h1 v-if="!header.title" class="channel__row-text">No title.</h1>
        <h1 v-else class="channel__row-text">{{ header.title }}</h1>
      </div>
      <div class="channel__row">
        <p class="channel__row-badge">Description:</p>
        <p v-if="!header.description" class="channel__row-text">
          No description.
        </p>
        <p v-else class="channel__row-text">{{ header.description }}</p>
      </div>
      <div class="channel__row">
        <p class="channel__row-badge">Last update:</p>
        <span v-if="!header.lastUpdate" class="channel__row-text">
          No update information.
        </span>
        <span v-else class="channel__row-text">{{ header.lastUpdate }}</span>
      </div>
      <div class="channel__row">
        <p class="channel__row-badge">RSS channel:</p>
        <span v-if="!header.feedUrl" class="channel__row-text">
          No RSS link.
        </span>
        <a
          v-else
          :href="header.feedUrl"
          class="channel__row-text channel__row-text--link"
          target="_blank"
          >Link to Channel >
        </a>
      </div>
      <div class="channel__row">
        <p class="channel__row-badge channel__row-badge--error">
          Click on the button to simulate an error of incorrect link to the RSS
          feed.
        </p>
        <button class="channel__row-btn" @click.prevent="this.error = true">
          Throw error
        </button>
      </div>
    </div>

    <form class="search-form">
      <input
        v-model="inputSearch"
        type="text"
        class="search-form__input"
        placeholder="Type search request"
      />
      <button class="search-form__btn" @click="searchHandler" type="submit">
        Search
      </button>
    </form>

    <ul class="cards">
      <li
        v-if="copyFeeds.length"
        v-for="feed in copyFeeds"
        class="cards__card"
        :key="feed.id"
      >
        <img
          v-if="!feed.image"
          src="../assets/img/placeholder-news.webp"
          alt="News Photo Placeholder"
          class="cards__card-image"
        />
        <img
          v-else
          :src="feed.image"
          alt="News Image"
          class="cards__card-image"
        />

        <div class="cards__card-body">
          <h3 v-if="!feed.title" class="cards__card-title">No information.</h3>
          <h3 v-else class="cards__card-title">
            {{ feed.title }}
          </h3>

          <p v-if="!feed.content" class="cards__card-content">No content.</p>
          <p v-else class="cards__card-content">
            {{ feed.content }}
          </p>
        </div>
        <div class="cards__card-footer">
          <span v-if="!feed.pubDate" class="cards__card-date">
            No punlication date.
          </span>
          <span v-else class="cards__card-date">
            {{ feed.pubDate }}
          </span>

          <span
            v-if="!feed.link"
            class="cards__card-link cards__card-link--error"
            >No link</span
          >
          <a v-else :href="feed.link" target="_blank" class="cards__card-link"
            >See more</a
          >
        </div>
      </li>
      <li v-else class="cards__card-no-results">
        Oops...There are no results :(
      </li>
    </ul>
  </div>

  <loader v-if="loading" />

  <div class="error" v-if="error" @click="resetGetData">
    <p class="error__info">There is a problem with the RSS channel.</p>
    <button class="error__btn">Reset URL</button>
  </div>
  
</template>

<script>
import Fuse from "fuse.js";
import Parser from "rss-parser/dist/rss-parser.js";
import Loader from "./Loader.vue";

export default {
  props: {
    url: {
      type: String,
      required: true,
    },
  },
  data: () => {
    return {
      header: {},
      feeds: [],
      copyFeeds: [],
      inputSearch: "",
      loading: false,
      error: false,
    };
  },

  name: "FeedPreview",

  components: { Loader },

  async created() {
    this.getData();
  },

  methods: {
    async getData() {
      const MY_PROXY = "https://rss-parser.vovaost.workers.dev/?";

      const defaultUrl = "https://www.gamespot.com/feeds/mashup/";

      const parser = new Parser({
        customFields: {
          item: [["media:content", "mediaContent", { keepArray: false }]],
        },
      });

      try {
        this.loading = true;
        const fetchFeeds = await parser.parseURL(
          `${MY_PROXY}${!this.error ? this.url : defaultUrl}`
        );

        this.feeds = fetchFeeds.items.map((item) => ({
          title: item.title,
          content: item.contentSnippet,
          image: item.mediaContent?.$.url,
          pubDate: item.pubDate,
          id: item.guid,
          link: item.link,
        }));

        this.copyFeeds = [...this.feeds];

        this.header = {
          title: fetchFeeds.title,
          description: fetchFeeds.description,
          lastUpdate: fetchFeeds.lastBuildDate,
          feedUrl: fetchFeeds.feedUrl,
        };
      } catch (error) {
        console.log("Error with fetchFeeds", error);
        this.error = true;
      } finally {
        this.loading = false;
      }
    },

    resetGetData() {
      this.getData();
      this.error = false;
    },

    searchHandler(e) {
      e.preventDefault();
      console.log("search");

      if (this.inputSearch === "") return (this.copyFeeds = [...this.feeds]);

      const options = {
        includeScore: true,
        keys: ["title"],
      };

      const fuse = new Fuse(this.feeds, options);

      this.copyFeeds = fuse
        .search(this.inputSearch)
        .map((el) => Object.assign(el.item));
    },
  },
};
</script>
