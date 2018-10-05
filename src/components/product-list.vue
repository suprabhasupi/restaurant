<template>
	<div class="wrapper">
	<div class="product" v-for="product in list" :key="product.name">
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
</template>

<script>
	export default {
		name: 'ProductList',
		data () {
			return {
			}
		},
		props: ['list'],
	}
</script>

<style>
.wrapper {
	width: 100%;
	display: flex;
	flex-wrap: wrap;
	justify-content: space-around;
}
.product {
	width: 30%;
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