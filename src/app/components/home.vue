<template>
  <div class="container-fluid home">
  <div v-show="error" class="alert alert-danger home__error-message" role="alert">
    {{errorMessage}}
  </div>
  <section class="row no-gutters input-group mb-3 home__search">
    <input
      type="text"
      class="form-control home__search__input"
      v-model="searchInput"
      placeholder="Buscar"
      aria-label="Search"
    ></input>
    <div class="input-group-append">
      <button
        class="btn fa fa-search home__search__button"
        type="button"
        @click="sortByWord"
      ></button>
    </div>
  </section>

  <section class="row no-gutters home__filters">
    <div class="home__filters__label">
      <label>Filtrar por:</label>
    </div>

    <div class="home__filters__button">
      <button @click="sortGames('popularity')" type="button" class="btn btn-info">
        Popularidade
      </button>
    </div>

    <div class="home__filters__button">
      <button @click="sortGames('viewers')" type="button" class="btn btn-info">
        Visualizações
      </button>
    </div>
  </section>

  <grid :games="games"></grid>

  <loader :active="loader"></loader>

  </div>
</template>
<script>
  import GamesService from '_services/games.js'
  import Normalize from '_helpers/normalize.js'
  import Helpers from '_helpers/helpers.js'
  import Grid from '_components/grid.vue'
  import Loader from '_components/loader.vue'
  import store from '_vuex/store'

  export default {
    name: 'Home',
    data() {
      return {
        searchInput: '',
        offset: 0,
        loader: false,
        fetching: false,
        error: false,
        errorMessage: ''
      }
    },
    mounted() {
      this.initObserveScroll()

      //call only the games has not loaded before
      //To offline navigation
      if(!this.games.length) {
        this.fetchGamesList()
      }

      if(this.searchInputState != '') {
        this.searchInput = this.searchInputState
        this.sortByWord()
      }
    },
    components: {
      Grid,
      Loader
    },
    methods: {
      toggleLoader() {
        this.loader = !this.loader;
      },
      fetchGamesList() {
        this.toggleLoader();

        GamesService.getList()
        .then((result) => {
          if(result.data) {
            this.toggleLoader();
            this.$store.dispatch('updateGameList', Normalize.normalizeGameArray(result.data.top))
          }

        }).catch((error) => {
          this.toggleLoader();
          this.errorHandle('Erro ao atualizar, verifique sua conexão com a internet.')
        })
      },
      incrementGamesList() {
        this.toggleLoader();
        this.fetching = true;
        GamesService.getList(this.games.length)
        .then((result) => {
          if(result.data) {
            this.toggleLoader();
            this.fetching = false;
            let newGames = Normalize.normalizeGameArray(result.data.top)
            this.$store.dispatch('updateGameList', Array.prototype.concat(this.games, newGames))
          }

        }).catch((error) => {
          this.toggleLoader();
          this.fetching = false;
          this.errorHandle('Erro ao atualizar, verifique sua conexão com a internet.')
        })
      },
      initObserveScroll() {
        window.addEventListener('scroll', (e) => {
          let scrolled = document.documentElement.scrollTop
          let endOfPage = document.body.offsetHeight - window.innerHeight

          //call if no fetching data is running
          if (scrolled >= endOfPage && !this.fetching) {
            this.incrementGamesList()
          }
        })
      },
      sortGames(criteria) {
        this.$store.dispatch('updateGameList', this.games.sort((a, b) => b[criteria] - a[criteria]))
      },
      sortByWord() {
        if(this.searchInput === '' && this.gamesTemp.length) {
          this.$store.dispatch('updateGameList', this.gamesTemp)
        }

        if(!this.gamesTemp.length) {
          this.$store.dispatch('updateGameTemp', this.games)
        }

        this.$store.dispatch('updateGameList', Helpers.filterName(this.searchInput, this.gamesTemp))
      },
      errorHandle(message) {
        this.error = true
        this.errorMessage = message
        document.querySelector('.home__error-message')
          .scrollIntoView({block: "end", behavior: "smooth"})

        setTimeout(() => {
          this.error = false
        }, 5000)
      }
    },
    watch: {
      searchInput() {
        this.$store.dispatch('updateSearch', this.searchInput)
        this.sortByWord()
      }
    },
    computed: {
      searchInputState: function () {
       return store.state.Search.data
      },
      games: function () {
       return store.state.Game.list
      },
      gamesTemp: function () {
       return store.state.Game.temp
      }
    }
  }
</script>
<style lang="sass" scoped>
  @import '~_sass/config';

  .home {
    padding-top: 10px;

    @media screen and(min-width: $screen-md) {
      padding-top: 20px;
      max-width: $screen-md;
      margin: 0 auto;
    }

    &__search {
      height: 40px;
      width: 100%;
      margin-bottom: 10px;

      &__input {
        flex: 1;
        background-color: $tertiary-color;

        &:focus {
          box-shadow: none;
        }
      }

      &__button {
        background-color: $tertiary-color;
        border: 1px solid $quaternary-color;
        border-left: none;

        &:focus {
          box-shadow: none;
        }

        &:before {
          font-size: $font-size-big;
          color: $secondary-color
        }
      }
    }

    &__filters {
      @media screen and(min-width: $screen-mc) {
        max-width: 300px;
      }

      &__button {
        margin-right: 5px;
        height: 40px;
        flex: 1;

        &:last-child {
          margin-right: 0;
        }
      }

      &__label {
        flex: 1;
        line-height: 2.5;
        font-weight: bold;
        font-size: $font-size-default;
        margin-right: 5px;
      }
    }
  }
</style>