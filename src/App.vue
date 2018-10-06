<template>
  <div id="app">
    <div class="error" v-if="!onlneState">You are offline. Please check your connectivity</div>
    <div class="header">
      <div class="input-wrapper">
        <input type="text"
          name="search"
          class='search'
          placeholder="Search for Restaurants"
          v-model="searchInput"
          @input="findRestaurant(searchInput)"
          @focus="findRestaurant(searchInput)"
          :aria-activedescendant="activedescendant"
          @keydown.down="onArrowDown"
          @keydown.up="onArrowUp"
          @keyup.enter="handleAutoCompleteSelection(searchResult[arrowCounter].name)"
         />
        <div v-if="isAutoCompleteVisible && searchResult.length > 0" class="auto">
          <ul>
            <li v-for="(resturant, i) in searchResult"
              :key="i"
              @click="handleAutoCompleteSelection(resturant.name)"
              :class="{ 'is-active': isSelected(i) }"
              :id="getId(i)"
              :aria-selected="isSelected(i)"
              >
              {{resturant.name}}
              <br />
              <span class="cuisines">{{resturant.cuisines.join(', ')}}</span>
            </li>
          </ul>
        </div>
        <div v-if="isRecentSearchVisible && recentSearch.length > 0" class="auto">
          <p class="recent-search">Recent Search:</p>
          <ul class="recent-search-list">
            <li v-for="(resturant, i) in recentSearch" :key="i" >
              <div @click="handleAutoCompleteSelection(resturant)" class="resturant-name">{{resturant}}</div>
              <div @click="removeRecentSearch(resturant)" class="remove-recent-search">&#x2715;</div>
            </li>
          </ul>
        </div>
      </div>
      <div class="sort-by">
        Sort By:
        <div @click="sortAccgToRating" :class="isActive === 'rating' ? 'active' : 'not-active'">Rating</div>
        <div @click="sortAccgToDeliveryTime" :class="isActive === 'delivery' ? 'active' : 'not-active'">Delivery Time</div>
      </div>
    </div>

    <div class="product-wrapper">
      <ProductList :list="restrauntList" v-if="!isSearching" />
      <ProductList :list="restrauntList" v-if="isSearching" />
      </div>
    </div>
</template>
<script>
import ProductList from './components/product-list.vue'
export default {
  name: 'App',
  data () {
    return {
      restrauntList: [],
      searchInput: '',
      searchResult: [],
      onlneState: true,
      isSearching: false,
      isActive: '',
      recentSearch: JSON.parse(localStorage.getItem('recentSearch') || '[]'),
      isAutoCompleteVisible: false,
      isRecentSearchVisible: false,
      arrowCounter: 0,
      activedescendant: ''
    }
  },
  components: {
    ProductList
  },
  methods: {
    getRestaurantList () {
      fetch('https://s3.ap-south-1.amazonaws.com/wc-search-widget/restaurants.json').then(function (res) {
        return res.json()
      }).then(res => {
        this.restrauntList = res
      })
    },
    handleAutoCompleteSelection (query) {
      this.searchInput = query
      if (!this.recentSearch.includes(query)) {
        this.recentSearch.push(query)
        localStorage.setItem('recentSearch', JSON.stringify(this.recentSearch))
      }
      this.findRestaurant(query)
      this.isAutoCompleteVisible = false
    },
    findRestaurant (query) {
      if (query.length === 0) {
        this.isSearching = false
        this.isRecentSearchVisible = true
      } else {
        this.isSearching = true
        this.isRecentSearchVisible = false
      }
      query = query.charAt(0).toUpperCase() + query.slice(1)
      if (query.length === 0) {
        this.searchResult = []
        this.isAutoCompleteVisible = false
      } else {
        this.isAutoCompleteVisible = true
        let matchingProducts = this.matchRestaurant(query)
        let matchingCuisines = this.matchCuisines(query)
        // combining matching cuisines and name then de-duplicating
        this.searchResult = [...new Set([...matchingProducts, ...matchingCuisines])]
      }
    },
    matchCuisines (input) {
      var reg = new RegExp(input.split('').join('\\w*').replace(/\W/, ''), 'i')
      return this.restrauntList.filter(resturant => {
        let isFound = resturant.cuisines.filter(cus => {
          if (cus.match(reg)) {
            return resturant
          }
        })
        if (isFound.length > 0) {
          return resturant
        }
      })
    },
    matchRestaurant (input) {
      var reg = new RegExp(input.split('').join('\\w*').replace(/\W/, ''), 'i')
      return this.restrauntList.filter(a => {
        if (a.name.match(reg)) return a
      })
    },
    handleClickOutside (evt) {
      let allowed = ['search', 'recent-search', 'remove-recent-search']
      if (!allowed.includes(evt.toElement.className)) {
        this.isAutoCompleteVisible = false
        this.isRecentSearchVisible = false
        this.arrowCounter = -1
      }
    },
    sortAccgToRating () {
      this.isActive = 'rating'
      this.restrauntList.sort((a, b) =>
        b.rating - a.rating)
    },
    sortAccgToDeliveryTime () {
      this.isActive = 'delivery'
      this.restrauntList.sort((a, b) =>
        a.delivery_time - b.delivery_time)
    },
    onArrowDown (evt) {
      if (this.isAutoCompleteVisible) {
        if (this.arrowCounter < this.searchResult.length - 1) {
          this.arrowCounter = this.arrowCounter + 1
          this.setActiveDescendent()
        }
      }
    },
    onArrowUp (evt) {
      if (this.isAutoCompleteVisible) {
        if (this.arrowCounter > 0) {
          this.arrowCounter = this.arrowCounter - 1
          this.setActiveDescendent()
        }
      }
    },
    setActiveDescendent () {
      this.activedescendant = this.getId(this.arrowCounter)
    },
    getId (index) {
      return `result-option-${index}`
    },
    isSelected (i) {
      return i === this.arrowCounter
    },
    removeRecentSearch (name) {
      this.recentSearch = this.recentSearch.filter(a => a !== name)
      localStorage.setItem('recentSearch', JSON.stringify(this.recentSearch))
    }
  },
  mounted () {
    this.getRestaurantList()
    this.onlneState = navigator.onLine
    document.addEventListener('click', this.handleClickOutside)
  },
  destroyed () {
    document.removeEventListener('click', this.handleClickOutside)
  }
}

</script>

<style>
#app {
  background: #f6f6f6;
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
  background: linear-gradient(-23deg, rgb(255, 116, 23), rgb(240, 94, 35));
  padding: 20px 165px;
}

.input-wrapper {
    flex: 3;
    position: relative;
}

input {
  width: 30%;
  padding: 8px;
  border-radius: 4px;
  border: none;
  display: flex;
  margin-left: 20px;
  outline: none;
  line-height: 24px;
  font-size: 12px;
}

.product-wrapper {
  width: 80%;
  margin: auto;
}

p {
  margin: 5px 0;
}

img {
  width: 100%;
}

.sort-by {
  align-items: center;
  display: flex;
  font-weight: 600;
  color: white;
}

.sort-by>div {
  cursor: pointer;
  margin: 0 10px;
  line-height: 25px;
  font-weight: 500;
}

.auto {
  position: absolute;
  top: 40px;
  width: 29.2%;
  left: 20px;
  background: white;
  padding: 10px;
  text-align: left;
  border: 1px solid #ccc;
  border-bottom-left-radius: 4px;
  border-bottom-right-radius: 4px;
}

.cuisines {
  color: #818181;
  font-size: 12px;
}

.resturant-name {
  flex: 1;
}

.auto > ul > li:last-child {
  border-bottom: none;
}
.error {
    color: white;
    background: #fa4141;
    font-size: 12px;
    padding: 5px;
    font-weight: 600;
}
ul {
  margin: 0;
  padding: 0;
  list-style: none;
}

li {
  border-bottom: 1px solid #dbd8d8;
  padding: 8px 12px;
  font-size: 14px;
  text-align: left;
  cursor: pointer;
}

.recent-search-list {
  font-size: 12px;
  color: #818181;
}

.recent-search-list > li {
  display: flex;
  justify-content: space-between;
}

.recent-search {
  color: #818181;
  font-weight: 500;
}

.remove-recent-search {
  /*font-size: 14px;*/
  line-height: 14px;
  width: 16px;
  display: flex;
  justify-content: center;
  cursor: pointer;
}

.is-active > .cuisines {
  color:white;
}
.active {
  /*font-weight: 600;*/
  border-bottom: 2px solid white;
}

.is-active {
  background: #ff891a;
  color: white;
  font-weight: 500;
  border-radius: 4px;
}

.not-active {
  font-weight: normal;
  border-bottom: 2px solid transparent;
}

@media screen and (max-width: 48em) {
  .product, .product-wrapper {
    width: 100%;
  }

  input {
    width: 85%;
  }

  .header {
    display: block;
    padding: 20px;
  }

  .sort-by {
    justify-content: center;
    padding-top: 12px;
  }

  .auto {
    width: 89%;
  }
}

</style>
