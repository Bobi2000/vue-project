<template>
  <transition name="slide-fade">
    <div v-if="show">
      <a class="animeLink" v-bind:href="'/anime/' + anime.categoryId">
        <sub-header v-bind:message="anime.categoryTitle"></sub-header>
      </a>
      <div class="container">
        <h1 class="my-4">
          {{anime.title}}
          <small></small>
        </h1>
        <div class="row">
          <div class="col-md-3">
            <h3 class="my-3"></h3>

            <img class="img-fluid" v-bind:src="anime.thumbnailURL" alt />
          </div>
          <div class="col-md-8">
            <h2>Score:</h2>
            <h2>{{ratings.avrRating}}</h2>
            <h2 v-if="notLoggedInAvrRating != 0">{{notLoggedInAvrRating}}</h2>
            <star-rating
              v-bind:increment="0.01"
              v-if="isUserLogged"
              v-bind:rating="ratings.avrRating"
              v-bind:read-only="true"
              v-bind:show-rating="false"
              v-bind:max-rating="10"
              v-bind:star-size="34"
            ></star-rating>
            <star-rating
              v-bind:increment="0.01"
              v-if="!isUserLogged"
              v-bind:rating="notLoggedInAvrRating "
              v-bind:read-only="true"
              v-bind:show-rating="false"
              v-bind:max-rating="10"
              v-bind:star-size="34"
            ></star-rating>
            <div v-if="!ratings.hasUserVoted && isUserLogged">
              <h2>Rate:</h2>
              <star-rating
                @rating-selected="setRating"
                v-bind:max-rating="10"
                v-bind:star-size="34"
                v-model="rating"
              ></star-rating>
            </div>

            <h3 class="my-3">Anime synopsis</h3>
            <p>{{anime.synopsis}}</p>
          </div>
        </div>
        <br />
        <br />
        <div class="row">
          <div class="col-md-10">
            <iframe width="840" height="630" v-bind:src="anime.trailerURL"></iframe>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import SubHeader from "../core/SubHeader.vue";
import StarRating from "vue-star-rating";
import authStore from "../../store/auth.js";
import AnimeService from "../services/anime.js";

const restAnimeService = new AnimeService();

export default {
  name: "AnimeDetails",

  components: {
    SubHeader,
    StarRating
  },
  data() {
    return {
      anime: {},
      ratings: {},
      rating: 0,
      notLoggedInAvrRating: 0,
      isUserLogged: false,
      show: false
    };
  },
  created() {},

  mounted() {
    this.loadAnimes(this.$route.params.id);
  },

  methods: {
    loadAnimes(id) {
      restAnimeService.getAnime(id).then(data => {
        this.anime = data.data;
        this.show = true;
        let isLogged = authStore.checkIfIsLogged();
        if (isLogged) {
          this.isUserLogged = true;
          let userId = authStore.getUserId();
          restAnimeService.getAnimeUserRating(id, userId).then(data => {
            this.ratings = data.data;
            this.show = true;
          });
        } else {
          restAnimeService.getAnimeRating(id).then(data => {
            this.notLoggedInAvrRating = data.data;
            this.show = true;
          });
        }
      });
    },
    setRating: function(rating) {
      this.rating = rating;
      let animeId = Number(this.$route.params.id);
      let userId = Number(authStore.getUserId());

      restAnimeService.setRating(rating, userId, animeId);
    }
  }
};
</script>

<style scoped>
figure {
  margin: 0;
  padding: 0;
  background: #fff;
  overflow: hidden;
}
figure:hover + span {
  bottom: -36px;
  opacity: 1;
}
.hover11 figure img {
  opacity: 1;
  -webkit-transition: 0.3s ease-in-out;
  transition: 0.3s ease-in-out;
}
.hover11 figure:hover img {
  opacity: 0.8;
}

.animeLink {
  text-decoration: none;
  color: black;
}

.slide-fade-enter-active {
  transition: all 1s ease;
}
.slide-fade-leave-active {
  transition: all 0.8s cubic-bezier(1, 0.5, 0.8, 1);
}
.slide-fade-enter, .slide-fade-leave-to
/* .slide-fade-leave-active below version 2.1.8 */ {
  transform: translateX(10px);
  opacity: 0;
}
</style>
