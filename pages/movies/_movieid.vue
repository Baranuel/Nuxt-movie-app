<template>
    <div class="flex items-center justify-center items-center h-screen w-screen   ">
        <div class="flex justify-center  w-full ">
            <img :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`" alt="">
            <div class="p-8 w-2/6 relative card">
            <div class="mt-6 flex justify-between items-end">
                <div class="items-start">
                    <p class="meta-tag">Title</p>
                    <h1 class="text-4xl">{{movie.original_title}}</h1>
                </div>
                <div class="items-start">
                    <p class="meta-tag">Rating</p>
                    <h1 class="text-4xl ">{{movie.vote_average}}/10</h1>
                </div>
                <div class="items-start">
                    <p class="meta-tag">vote count</p>
                    <h3 class="text-4xl ">{{movie.vote_count}}</h3>
                </div>
            </div>
            <div class="mt-4 ">
                <p class="meta-tag">Genres</p>
                <div class="flex">
                 <p v-for="genre in movie.genres" :key="genre.id" class="text-sm mr-2">{{genre.name}},</p>
                </div>
            </div>
            <div class="mt-4 ">
                <p class="meta-tag">Languages</p>
                <div class="flex">
                 <p v-for="language in movie.spoken_languages" :key="language.name" class="text-sm mr-2">{{language.english_name}},</p>
                </div>
            </div>
            <div class="mt-4 ">
                <p class="meta-tag">Description</p>
                <div class="flex">
                 <p>{{movie.overview}} </p>
                </div>
            </div>
            <div class="mt-4 ">
                <p class="meta-tag">Random Keywords</p>
                <div class="flex">
                 <p class="mr-2" v-for="keyword, index in keywordArr" :key="index">{{keyword.name}}</p>
                </div>
            </div>
            <div class="similarMovies ">
            <p class="meta-tag">Similar Movies</p>
            <button v-if="amountShown > 3" @click="amountShown = 3,removeClass()">Hide</button>
                <div v-if="amountShown <= 3" class=" flex w-full flex-wrap ">
                <div  v-for="movie,index in similarMovies" :key="index">
                <NuxtLink :to="{ name: 'movies-movieid', params: { movieid: movie.id } }">
                    <img v-if="index <= amountShown " class="image h-40 m-2" :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`">
                </NuxtLink>
                </div>
                </div>
            <div v-else class=" flex w-full flex-wrap ">
            <div  v-for="movie,index in similarMovies" :key="index">
            <NuxtLink :to="{ name: 'movies-movieid', params: { movieid: movie.id } }">
                <img v-if="index <= amountShown " class="image h-40 m-2" :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`">
            </NuxtLink>
            </div>
            </div>

            <button v-if="amountShown <= 3" @click="amountShown = 20, addClass()">see more</button>
            </div>
            <button @click="addRating()">add rating</button>
            <button @click="getRatedMovies()">get rated</button>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios'
export default {
name:"movie",
data(){
    return{
        movie:{},
        movieId:"",
        randomKeyWord1:'',
        randomKeyWord2:'',
        keywordArr: [],
        similarMovies: [],
        amountShown: 3,
        validSession:''
    }
},
async fetch(){
    this.movieId = this.$route.params.movieid

    const data = await axios.get(`https://api.themoviedb.org/3/movie/${this.movieId}`, {
        params:{
            api_key:'3fc22e7d1cf6b4347458965b52749b53'
        }
    })
        console.log(data.data)
        this.movie = data.data
        console.log(this.movie)
},
async mounted(){
           await this.getSession(),
           await  this.getKeyWords()
           await  this.getSimilar()
},
methods:{
     async getKeyWords(){
         const data = axios.get(`https://api.themoviedb.org/3/movie/${this.movieId}/keywords`, {
             params:{
                 api_key:'3fc22e7d1cf6b4347458965b52749b53'
             }
         })

         const result = await data

            for( let i = 0; i< result.data.keywords.length ; i++){
                // let randomIndex = Math.floor(Math.random() * result.data.keywords.length )
                this.randomKeyWord1= result.data.keywords[ Math.floor(Math.random() * result.data.keywords.length )]
                this.randomKeyWord2= result.data.keywords[ Math.floor(Math.random() * result.data.keywords.length )]
            }
            this.keywordArr.push(this.randomKeyWord1,this.randomKeyWord2)

    },
    async getSimilar(){
        const data = axios.get(`https://api.themoviedb.org/3/movie/${this.movieId}/similar`,{
            params:{
                api_key:'3fc22e7d1cf6b4347458965b52749b53',
                page:100
            }
        })
        const result = await data
        result.data.results.map(similarMovie => {
            this.similarMovies.push(similarMovie)
        })

    },
    addClass(){
        const similarMovies = document.querySelector('.similarMovies')
        similarMovies.classList.add('similarMoviesSeeMore')
    },

    removeClass(){
        const similarMovies = document.querySelector('.similarMovies')
        similarMovies.classList.remove('similarMoviesSeeMore')
    },
       async addRating(){
           const session = window.localStorage.getItem('session')
           console.log(window.localStorage.getItem('session'))
           console.log(session)
        const rating = axios.post(`https://api.themoviedb.org/3/movie/${this.movieId}/rating`,
             {
                 value:9
             }, 
            {
           params:{
               api_key:'3fc22e7d1cf6b4347458965b52749b53',
               guest_session_id: session
           },
        })
        .then((response) =>{
            console.log(response)
        },
        (error) =>{
            console.log(error)
        },
        console.log(this.movie));    
},
async getSession(){
},
 async getRatedMovies(){
   const session = window.localStorage.getItem('session')
     const data = axios.get(`https://api.themoviedb.org/3/guest_session/${session}/rated/movies`,{
        params:{
           api_key:'3fc22e7d1cf6b4347458965b52749b53',
           sort_by: 'created_at_asc'
        }
     })
     const result = await data
     console.log(result)
}
}

}
</script>

<style lang="scss">
.card{
    background-color:#0f0f14
}
.meta-tag{
    color:#c40c40
}
.similarMovies{

}
.similarMoviesSeeMore{
    position:absolute;
    background-color:#0f0f14;
    top:0;
    margin:0 auto;
    height:100%;
    width:100%;
    overflow-y:scroll;
}
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.1s ease;
}

.fade-enter-from{
    opacity:0
};
.fade-enter-to{
    opacity:1
}
.fade-leave-from{
    opacity:1
}
.fade-leave-to {
  opacity: 0;
}
</style>