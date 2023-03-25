<template>
  <div class="section">
    <div class="container">
      <div class="searchbar">
        <form @submit.prevent = "fetchSearchNews">
          <input type="text" placeholder="search..." v-model="searchword" />
        </form>
        <div class="search-icon">
          <i v-if="!isBusy" class="fas fa-search" @click="fetchSearchNews"></i>
          <i v-else class="fas fa-spinner fa-spin"></i>
          <i class="fas fa-times" @click="fetchTopNews"></i>
        </div>
      </div>
      <div class="result-list">
        <article v-for="(article, index) in articles" :key="index" @click="navTo(article.url)">
          <header>
            <img v-if="article.urlToImage" :src="article.urlToImage" alt="" />
            <i v-else class="fas fa-image"></i>
          </header>
          <section v-html="article.title"></section>
          <footer>
            <i class="fas fa-chevron-right"></i>
          </footer>
        </article>
      </div>
      <div ref="infinitescrolltrigger" id="scroll-trigger">
        <i v-if="showLoader" class="fas fa-spinner fa-spinn"></i>
      </div>
    </div>
  </div>
</template>

<script>
// import { runInThisContext } from "vm";

export default {
  name: "NewsComp",
  // props: [
  //   "apiKey" = d3c81fdc4f0b4c62aec485c3448833f3,
  // ],
  data: () => {
    return {
      apiUrl: "",
      isBusy: false,
      showLoader: false,
      currentPage: 1,
      totalResults: 0,
      maxPerPage: 20,
      searchword: "",
      articles: [],
      country: "us",
      apiKey: "d3c81fdc4f0b4c62aec485c3448833f3",
    };
  },
  computed: {
    pageCount(){
      return Math.ceil(this.totalResults/this.maxPerPage)
    }
  },
  methods: {
    navTo(url){
      window.open(url);
    },
    resetData() {
      this.currentPage = 1;
      this.articles = [];
    },
    fetchSearchNews(){
      if(this.searchword !== ""){
        this.apiUrl =
          "https://newsapi.org/v2/everything?q=" +
          this.searchword +
          "&pageSize=" +
          this.maxPerPage +
          "&apiKey=" +
          this.apiKey;
  
          this.isBusy = true
  
          this.resetData();
          this.fetchData();
      }else{
        this.fetchTopNews();
      }
    },
    fetchTopNews() {
      this.apiUrl =
        "https://newsapi.org/v2/top-headlines?country=" +
        this.country +
        "&pageSize=" +
        this.maxPerPage +
        "&apiKey=" +
        this.apiKey;
      this.isBusy = true;
      this.searchword = "";

      this.resetData();
      this.fetchData();
    },
    fetchData() {
      let req = new Request(this.apiUrl + "&page=" + this.currentPage);
      fetch(req)
        .then((resp) => resp.json())
        .then((data) => {
          this.totalResults = data.totalResults;
          data.articles.forEach((element) => {
            this.articles.push(element);
          });
          this.isBusy = false;
          this.showLoader = false;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    scrollTrigger(){
      const observer = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
          if(entry.intersectionRatio > 0 && this.currentPage < this.pageCount){
            this.showLoader = true;
            this.currentPage += 1;
            this.fetchData();
          }
        });
      });

      observer.observe(this.$refs.infinitescrolltrigger)
    }
  },
  created() {
    this.fetchTopNews();
  },
  mounted(){
    this.scrollTrigger();
  }
};
</script>

<style scoped>
.section {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
.section .container {
  width: 80%;
}

.searchbar {
  display: flex;
  justify-content: space-between;
  width: 100%;
  height: 60px;
  font-size: 1.6rem;
  background-color: #222;
}

.searchbar form {
  width: 100%;
}

.searchbar input {
  padding: 0;
  margin: 0;
  width: 80%;
  height: 60px;
  border: none;
  font-size: 2rem;
  color: #fff;
  background-color: #222;
}

.searchbar input:focus {
  outline: none;
}

.search-icon {
  width: 20%;
  display: flex;
  color: #fff;
  /* justify-content: center; */
  align-items: center;
  padding-right: 10px;
}

.search-icon i {
  margin-left: 15px;
  cursor: pointer;
}

.result-list {
  padding-top: 60px;
}

.result-list article {
  display: grid;
  grid-template-columns: 200px auto 40px;
  grid-template-rows: 100px;
  border-bottom: 1px solid #ccc;
  overflow: hidden;
  cursor: pointer;
}

article header {
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
}

header img {
  max-width: 100%;
  height: auto;
}

header i {
  font-size: 2rem;
}

article section {
  margin: 0;
  padding: 10px;
  height: 100px;
}

article footer {
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 1.6rem;
  color: #888;
}

#scroll-trigger{
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100px;
  font-size: 1.6rem;
}
</style>
