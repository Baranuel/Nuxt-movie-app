<template>
  <div>
    <Hero />
    <div class="container search">
      <input
        @keyup.enter="$fetch"
        type="text"
        placeholder="Search"
        v-model.lazy="searchInput"
      />
      <button @click="searchByGenres()">search</button>
      <button @click="getRatedMovies()">get Rated movies</button>
      <button @click="clearSearch()" v-if="searchInput !== ''" class="button">Clear</button>
    </div>

    <Loading v-if="$fetchState.pending"/>

    <div class="container movies">
      <div v-if="this.searchInput !== ''" id="movie-grid" class="movies-grid">
        <div v-for="movie in searchedMovies" :key="movie.id" class="movie">
          <div class="movie-img">
            <NuxtLink
              :to="{ name: 'movies-movieid', params: { movieid: movie.id } }"
            >
              <img
                :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`"
                :alt="`${movie.original_title}`"
              />
              <p class="review">{{ movie.vote_average }}</p>
              <p class="overview">{{ movie.overview }}</p>
            </NuxtLink>
          </div>
          <div class="info">
            <p class="title">
              {{ movie.title.slice(0, 25)
              }}<span v-if="movie.title.length > 25">...</span>
            </p>
            <p class="release">
              Released:
              {{
                new Date(movie.release_date).toLocaleString("en-us", {
                  month: "long",
                  day: "numeric",
                  year: "numeric",
                })
              }}
            </p>
            <NuxtLink
              class="button"
              :to="{ name: 'movies-movieid', params: { movieid: movie.id } }"
            >
              Get more info</NuxtLink
            >
          </div>
        </div>
      </div>
      <div v-else id="movie-grid" class="movies-grid">
        <div v-for="movie in movies" :key="movie.id" class="movie">
          <div class="movie-img">
            <NuxtLink
              :to="{ name: 'movies-movieid', params: { movieid: movie.id } }"
            >
              <img
                :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`"
                :alt="`${movie.original_title}`"
              />
              <p class="review">{{ movie.vote_average }}</p>
              <p class="overview">{{ movie.overview }}</p>
            </NuxtLink>
          </div>
          <div class="info">
            <p class="title">
              {{ movie.title.slice(0, 25)
              }}<span v-if="movie.title.length > 25">...</span>
            </p>
            <p class="release">
              Released:
              {{
                new Date(movie.release_date).toLocaleString("en-us", {
                  month: "long",
                  day: "numeric",
                  year: "numeric",
                })
              }}
            </p>
            <NuxtLink
              class="button"
              :to="{ name: 'movies-movieid', params: { movieid: movie.id } }"
            >
              Get more info</NuxtLink
            >
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Hero from "../components/Hero.vue";
import axios from "axios";
import Loading from "../components/Loading.vue";
export default {
  name: "IndexPage",
  layout: "default",
  components: { Hero, Loading },
  data() {
    return {
      movies: [],
      apiKey: "3fc22e7d1cf6b4347458965b52749b53",
      searchedMovies: [],
      searchInput: "",
      validSession:''
    };
  },
  async fetch() {
    if (this.searchInput === "") {
      await this.getMovies();
      return;
    }
    await this.searchMovies();
  },
  mounted(){
    this.getSession()
  },
  fetchDelay:500,
  methods: {
    async getMovies() {
      const data = axios.get("https://api.themoviedb.org/3/movie/now_playing", {
        params: {
          api_key: "3fc22e7d1cf6b4347458965b52749b53",
        },
      });
      const result = await data;
      result.data.results.map((movie) => {
        this.movies.push(movie);
      });
    },
    async searchMovies() {
      const data = axios.get("https://api.themoviedb.org/3/search/movie", {
        params: {
          api_key: "3fc22e7d1cf6b4347458965b52749b53",
          query: this.searchInput,
        },
      });
      const result = await data;
      result.data.results.map((movie) => {
        this.searchedMovies.push(movie);
      });
    },
   async searchByGenres(){
      const data = axios.get('https://api.themoviedb.org/3/genre/movie/list',{
        params:{
          api_key:'3fc22e7d1cf6b4347458965b52749b53'
        }
      })
      const result = await data;
        console.log(this.validSession)
    },
    clearSearch(){
      this.searchInput = '',
      this.searchedMovies = []
    },
    async getSession(){
    const data = axios.get('https://api.themoviedb.org/3/authentication/guest_session/new', {
        params:{
            api_key:'3fc22e7d1cf6b4347458965b52749b53'
        }
    })
    const result = await data
    const session = result.data.guest_session_id
    window.localStorage.setItem('session',session)
    console.log(window.localStorage.getItem('session'))
}
  },
};
</script>

<style lang="scss">

.loading{
  padding-top:120px;
  align-items: flex-start;
}

.search {
  padding: 1rem;

  input {
    padding: 0.5rem 2rem;
    color: black;

    &:focus {
      outline: rgb(133, 9, 25);
    }
  }
}

.movies {
  margin: 0 auto;
  padding: 1rem;

  .movies-grid {
    display: grid;
    grid-template-columns: 1fr;
    grid-template-rows: auto;
    @media (min-width: 500px) {
      grid-template-columns: repeat(2, 1fr);
    }
    @media (min-width: 750px) {
      grid-template-columns: repeat(2, 1fr);
    }
    @media (min-width: 1100px) {
      grid-template-columns: repeat(3, 1fr);
    }
    gap: 3rem;
  }
  .movie {
    width: 100%;

    .movie-img {
      position: relative;
      overflow: hidden;

      &:after {
        content: "";
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        transform: translateY(100%);
        transition: all 0.25s ease;
        background-image: linear-gradient(
          to top,
          rgb(133, 9, 25),
          rgba(0, 0, 0, 0.15)
        );
        opacity: 0;
      }

      &:hover {
        cursor: pointer;
        &:after {
          pointer-events: none;
          opacity: 1;
          transform: translateY(0);
        }
        .overview,
        .review {
          opacity: 1;
          text-shadow: 1px 2px black;
        }
        .overview {
          transform: translateY(0);
        }
      }

      .overview {
        position: absolute;
        bottom: 0;
        padding: 2rem;
        transition: all 0.25s ease;
        z-index: 100;
        transform: translateY(100%);
        opacity: 0;
      }
      .review {
        position: absolute;
        top: 0;
        left: 0;
        z-index: 100;
        opacity: 1;
      }
    }
  }
}
</style>
