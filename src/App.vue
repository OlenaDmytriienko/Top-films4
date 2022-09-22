<script setup>
import '../node_modules/reset-css'
import './style/index.scss'
// import DayFilm from '@/components/DayFilm/Film.vue'
import gif from '@/assets/1479.gif'
import img from '@/assets/cinema.png'
</script>

<template>

    <header class="container ">
      <div class="header header-content">
        <a href="./index.html" class="header__logo">Top Films</a>
         <select name="" id="year1" class="header__year1 action" v-model="simbolYearFrom" @change="onChangeYearFrom($event)">
           <option value=""></option>
           <option v-for="n in 100" v-bind:value="n+1922" v-bind:key="n+1922">{{n+1922}}</option>
        </select>
        <select name="" id="year" class="header__year action" v-model="simbolYear" @change="onChangeYear($event)">
           <option value=""></option>
          <option value="RATING">По рейтингу</option>
          <option value="YEAR">По году</option>
        </select>
        <select name="" id="genre" class="header__genre action" v-model="simbol" @change="onChange($event)">
          <option value="1" @click="AP()">Tриллер</option>
          <option value="2" @click="AP()">Драма</option>
          <option value="3" @click="AP()">Криминал</option>
          <option value="4">Мелодрама</option>
          <option value="5">Детектив</option>
          <option value="6">Фантастика</option>
          <option value="7">Приключения</option>
        </select>
        <span></span>
        <form >
          <input type="text" class="header__search" placeholder="Search" @input="searchValue($event)"   v-model="form.searchQuery"/>
        </form>
      </div>
    </header>
      <div class="container today"> 
      <div class="movie__today">
       <div class="movie1"> 
            <img :src="restaurant.posterUrlPreview" :alt="restaurant.nameRu" class="movie__cover1" />
          <div class="movie__info">
            <div class="movie__t">{{restaurant.nameRu}}  </div>
            <p class="modal__year"> {{restaurant.year}} </p>
          <div class="movie__c" v-for="(genre) in restaurant.genres" v-bind:key="genre.genre"> {{ genre.genre }}</div>

        </div>
</div>

  <!-- <DayFilm id="film.kinopoiskId " src="film.posterUrlPreview" alt="film.nameRu" yaer="film.year" title="" genre="genre.genre " />    -->

      </div>
    </div>
    <div class="container main" > {{searchQuery}}
      <div class="loading__box">
             <span  v-show="loading1"> 
          <img class="loading" :src="gif" alt="loading">
           </span></div>
      <div class="movies" >
        <div class="movie" v-for="(item, index) in restaurants" @click="Details(item.kinopoiskId )" v-bind:key="item.kinopoiskId"> 
          <div class="movie__cover-inner" @click="Details(item.filmId )" >
            <img  :src="getMissingImg(index)" :alt="getMissingName(index)" class="movie__cover" />
            <div class="movie__cover--darkened"></div>
          </div>
          <div class="movie__info">
            <div class="movie__title">{{getMissingName(index)}} </div>
            <div class="movie__category" > {{ item?.genres?.map(genres => genres?.genre).join(', ') ?? ' ' }}</div>
           <div v-if="item?.rating >= 7"  class="movie__average movie__average--green"> {{item?.rating}} </div>
            <div v-else-if="item?.rating > 5"  class="movie__average movie__average--orange"> {{item?.rating}} </div>
             <div v-else-if="item?.rating <= 5"  class="movie__average movie__average--red"> {{item?.rating}} </div>
                        <div v-if="item?.ratingKinopoisk >= 7"  class="movie__average movie__average--green"> {{item?.ratingKinopoisk}} </div>
            <div v-else-if="item?.ratingKinopoisk > 5"  class="movie__average movie__average--orange"> {{item?.ratingKinopoisk}} </div>
             <div v-else-if="item?.ratingKinopoisk <= 5"  class="movie__average movie__average--red"> {{item?.ratingKinopoisk}} </div>
          </div>
        </div>
      </div>
    </div>

    <div id="myModal" class="modal" v-show="visible">
      <span
        class="close"
         v-on:click="visible=false"
        >&times;</span>
        <div class="modal__container">
      <img class="modal-content" id="img01" :src="modalRestaurant.posterUrlPreview" :alt="modalRestaurant.nameOriginal"/>
      <div class="modal__box"> 
      <h4 class="modal__title"> {{modalRestaurant.nameOriginal}}</h4>
      <h5 class="modal__genre" v-for="(genre) in modalRestaurant.genres" v-bind:key="genre"> {{ genre.genre }} </h5>
      <p class="modal__year"> {{ modalRestaurant.year}} </p>
      <div class="modal__text"> {{modalRestaurant.description}} </div>
      <p class="modal__pg"> {{modalRestaurant.ratingAgeLimits}} </p></div>
    </div>
    </div> <div class="loading__box">
       <span  v-show="loading"> 
          <img class="loading" :src="gif" alt="loading">
           </span></div>
    <button class="next__page" @click="popular()"> <span @click="AP()">Show more</span></button>



</template>




<script>
export default {
  name: 'App',
  data () {
    return {
      api: 'https://kinopoiskapiunofficial.tech/api/v2.2/films/top?type=TOP_100_POPULAR_FILMS',
      out: '',
      simbol: '',
      simbolYear: '',
      simbolYearFrom: '',
      form:{searchQuery:""},
      searchQuery: '',
      page: 1,
      pageString: '&page=',
      order:'',
      orderString: 'order=',
      genres: '',
      genresString: 'genres=',
      yearFrom: '',
      yearFromString: 'yearFrom=',
      yearToString: 'yearTo=',
      defaultImg: img,
      apiArray: [],
      restaurants: [],
      restaurant: [],
      modalRestaurant: [],
      error: null,
      visible: false,
      loading:false,
      loading1:false,
      headers: {'Content-Type': 'application/json', "X-API-KEY": '236d1c9c-2de0-4ab3-a57f-f871ba0334a2'},
    }
  },
  methods: {
      AP(){
    // this.loading = true;
    // axios.get(this.api).then(({response})=>{
    //     this.loading=false;
    // }).catch((error)=>{
    //     console.log(error);
    //     this.loading=false;
    // });
  },
    async popular() {
      console.log(this.page);
      this.page++
      if(this.apiArray.length === 0 ) {
        await this.filter(this.api + this.pageString + this.page.toString())
      } else {
           await this.filterGenres(this.api + this.pageString + this.page.toString())
      }
      console.log(this.api + this.pageString + this.page.toString());
    },
      async SerchFilms(nameRu) {
      this.page = 1
      this.api = 'https://kinopoiskapiunofficial.tech/api/v2.1/films/search-by-keyword?keyword=' + nameRu
      await this.filter(this.api)
    },
    async Details(filmId) {
      if(filmId != undefined) {    this.visible = true
      await this.modal('https://kinopoiskapiunofficial.tech/api/v2.2/films/' + filmId)}
    },
    async FilterGenres() {
      this.page = 1
      this.apiArray = []
      if (this.genres != ''){
        this.apiArray.push(this.genresString + this.genres)
        // this.api = this.genresString + this.genres //записали genres=1
      } 
      if (this.order != ''){
         this.apiArray.push(this.orderString + this.order)
        //this.api = this.orderString + this.order //записали order=rating
      }
       if (this.yearFrom != ''){
         this.apiArray.push(this.yearFromString + this.yearFrom + "&" + this.yearToString + this.yearFrom)
        //this.api = this.orderString + this.order //записали order=rating
      }
      // if (this.genres != '' && this.order != ''){
      //   this.api = this.genresString + this.genres + "&" + this.orderString + this.order
      // } // записали genres=1&order=rating
      this.api = 'https://kinopoiskapiunofficial.tech/api/v2.2/films?' + this.apiArray.join('&')
      await this.filterGenres(this.api)
       this.loading=false;
    },
    async filter(api) {
      this.loading = true;
      console.log(api);
      try {
        const response = await fetch(api, {
          method: 'GET',
          headers: this.headers,
        }).then(this.checkStatus)
            .then(this.parseJSON);
            
        console.log(response)
        if (this.page == 1) {
          this.restaurants = response.films} 
          else {
            this.restaurants = this.restaurants.concat(response.films)
          }
         this.loading=false;

      } catch (error) {
        this.error = error
      }
    },
    async modal(api) {
      try {
        const response = await fetch(api, {
          method: 'GET',
          headers: this.headers,
        }).then(this.checkStatus)
            .then(this.parseJSON);
        // console.log(response)
       console.log(response.description);
        this.modalRestaurant = response
      } catch (error) {
        this.error = error
      }
    },
    async filterGenres(api) {
      this.loading = true;
      try {
        const response = await fetch(api, {
          method: 'GET',
          headers: this.headers,
        }).then(this.checkStatus)
            .then(this.parseJSON);
        console.log(response)
       if (this.page == 1) {
          this.restaurants = response.items} 
          else {
            this.restaurants = this.restaurants.concat(response.items)
          }
          this.loading=false;
          this.loading1 = false;
      } catch (error) {
        this.error = error
        this.loading = false;
      }
    },
 
    parseJSON: function (resp) {
      return (resp.json ? resp.json() : resp);
    },
    checkStatus: function (resp) {
      if (resp.status >= 200 && resp.status < 300) {
        return resp;
      }
      return this.parseJSON(resp).then((resp) => {
        throw resp;
      });
    },
          onChange(event) {
            this.simbol = event.target.value
            console.log(event.target.value)
            this.genres = event.target.value;
           this.FilterGenres()
           this.loading1 = true;
        },
        onChangeYear(event) {
            this.simbolYear = event.target.value
            console.log(event.target.value)
            this.order = event.target.value;
           this.FilterGenres()
           this.loading1 = true;
        },
         onChangeYearFrom(event) {
            this.simbolYearFrom = event.target.value
            console.log(event.target.value)
            this.yearFrom = event.target.value;
           this.FilterGenres()
           this.loading1 = true;
        },
          searchValue(event) {
            this.searchQuery = event.target.value
            console.log(event.target.value)
            
           this.SerchFilms(event.target.value)
},
        drawItems(event) {
          this.popular(event.page++)
        },
            getMissingImg(index){
return  this.restaurants[index]?.posterUrlPreview ?? this.defaultImg
      //  return require(this.restaurants[index].posterUrlPreview);
      },
           getMissingName(index){
return  this.restaurants[index]?.nameRu ?? ' '
      //  return require(this.restaurants[index].posterUrlPreview);
      },
      getMissingGenres(index){
return  this.restaurants[index]?.nameRu ?? ' '
      //  return require(this.restaurants[index].posterUrlPreview);
      },
  },

  async mounted () {
      this.loading1=true;
    try {
      const response = await fetch(this.api, {
        method: 'GET',
        headers: this.headers,
      }).then(this.checkStatus)
          .then(this.parseJSON);
      console.log(response)
console.log(Math.floor(Math.random() * (100 - 0 + 1)) + 1);
      this.restaurants = response.films
      this.restaurant = response.films[Math.floor(Math.random() * (20 - 1 + 1)) + 1]
      // console.log();
       this.loading1=false;
    } 
    catch (error) {
      this.error = error
      this.loading1=false;
    }
  },
  
}
</script>