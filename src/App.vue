<template>
  <div id="app">
    <div class="error" v-if="!onlneState">You are offline. Please check your connectivity</div>
    <div class="header">
      <input type="text" name="search" placeholder="Search for Restaurants" v-model="searchInput" @input="findRestaurant(searchInput)" />
      <div v-if="searchResult.length" class="auto">
        {{searchResult[0].name}}
      </div>
      <div class="sort-by">
        Sort By:
        <div @click="sortAccgToRating">Rating</div>
        <div @click="sortAccgToDeliveryTime">Delivery Time</div>
      </div>
    </div>
    <div class="product-wrapper">
      <div class="product" v-for="product in restrauntList" :key="product.name">
        <div class="img">
          <img :src="product.image">
      </div>
          <div class="info-wrapper">
            <div class="restro-info">
              <p>{{product.name}}</p>
              <div class="cuisines" v-if="product.cuisines">
                {{product.cuisines.join(', ')}}
              </div>
            </div>
            <hr>
            <div class="restro-tim">
              <p>{{product.rating}} </p>
              <p>{{product.delivery_time}} MINS</p>
            </div>
          </div>
        </div>
      </div>
      <!-- <router-view/> -->
    </div>
</template>
<script>
export default {
  name: 'App',
  data() {
    return {
      restrauntList: [],
      searchInput: '',
      searchResult: [],
      onlneState: true
    }
  },
  methods: {
    getRestaurantList() {
      fetch('https://s3.ap-south-1.amazonaws.com/wc-search-widget/restaurants.json').then(function(res) {
        return res.json()
      }).then(res => {
        this.restrauntList = res
        console.log('res---->>>', this.restrauntList)
      })
    },
    findRestaurant(query) {
      if (query.length === 0) {
        this.searchResult = []
      } else {
        let matchingProducts = this.matchRestaurant(query)
        this.searchResult = matchingProducts
      }
    },
    matchRestaurant(input) {
      return this.restrauntList.filter(a => {
        if (a.name.match(input)) {
          return a
        }
      })
    },
    sortAccgToRating() {
      console.log('sortAccgToRating', this.restrauntList)
      let b = this.restrauntList.sort((a, b) =>
        b.rating - a.rating)
    },
    sortAccgToDeliveryTime() {
      console.log('sortAccgToRating', this.restrauntList)
      let b = this.restrauntList.sort((a, b) =>
        a.delivery_time - b.delivery_time)
    }
  },
  mounted() {
    this.getRestaurantList()
    this.onlneState = navigator.onLine
    console.log('onlneState------>>>>>', this.onlneState)
  }
}

</script>
<style>
#app {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

.header {
  display: flex;
  justify-content: space-between;
  background: orange;
  padding: 10px;
}

input {
  width: 25%;
  padding: 8px;
  border: none;
  display: flex;
  margin-left: 20px;
  outline: none;
}

p {
  margin: 5px 0;
}

.product {
  width: 40%;
  display: flex;
  background: white;
  box-shadow: 0 1px 5px #d2cfcf;
  margin: 10px 0;
}

.img {
  width: 150px;
}

img {
  width: 100%;
}

.info-wrapper {
  flex: 1;
  text-align: left;
  padding: 0 20px;
}

.restro-tim {
  display: flex;
  justify-content: space-around;
}

.cuisines {
  display: flex;
}

.product-wrapper {
  padding: 15px;
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;
}

.sort-by {
  display: flex;
  font-weight: 600;
}

.sort-by>div {
  padding: 0 10px;
  font-weight: 500;
}

.auto {
  position: absolute;
  top: 40px;
  width: 24%;
  left: 30px;
  background: white;
  padding: 10px;
  border: 1px solid #ccc;
}
.error {
    color: white;
    background: #fa4141;
    font-size: 12px;
    padding: 5px;
    font-weight: 600;
}

@media screen and (max-width: 48em) {
  .product {
    width: 100%;
  }

  input {
    width: 85%;
  }

  .header {
    display: block;
  }

  .sort-by {
    justify-content: center;
    padding-top: 12px;
  }

  .auto {
    width: 80%;
  }
}

</style>
