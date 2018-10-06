<template>
  <div class="wrapper">
  <div class="product" v-for="product in list" :key="product.name">
    <div class="img">
      <img :src="product.image">
    </div>
      <div class="info-wrapper">
        <div class="restro-info">
          <p><b>{{product.name}}</b></p>
          <div class="cuisines" v-if="product.cuisines">
            {{product.cuisines.join(', ')}}
          </div>
        </div>
        <hr>
        <div class="restro-tim">
          <p class="rating"><span>&#x2605;</span>{{product.rating}} </p>
          <p>{{product.delivery_time}} Mins</p>
        </div>
      </div>
    </div>
    </div>
</template>

<script>
export default {
  name: 'ProductList',
  data () {
    return {
    }
  },
  props: ['list']
}
</script>

<style>
.wrapper {
  width: 100%;
  display: flex;
  flex-wrap: wrap;
  justify-content: flex-start;
  padding: 20px;
}
.product {
  width: 30%;
  display: flex;
  background: white;
  border-radius: 4px;
  box-shadow: 0 1px 5px #d2cfcf;
  margin: 10px 15px;
}

.img {
  width: 150px;
}

img {
  width: 100%;
  height: 100%;
  border-top-left-radius: 4px;
  border-bottom-left-radius: 4px;
}
.info-wrapper {
  flex: 1;
  text-align: left;
  padding: 0 20px;
}

hr {
  margin: 5px 0;
  border: none;
  border-bottom: 1px solid #dcdada;
}

.restro-tim {
  display: flex;
  justify-content: space-around;
}

.cuisines {
  display: flex;
  color: #818181;
  font-size: 12px;
}
.rating {
  display: flex;
  align-items: center;
}

.rating > span {
  color: #f97c27;
  font-size: 14px;
  padding-right: 8px;
}

@media screen and (max-width: 48em) {
  .product {
    width: 100%;
  }
  .wrapper {
    padding: 0;
  }
}
</style>
