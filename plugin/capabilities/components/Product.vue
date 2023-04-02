<template>
  <div v-bind:id="id" class="product">
    <div class="product-title">
      <span>{{ title }}</span>
    </div>
    <div v-bind:id="container_id" class="product-container">
      <product-capability
        v-for="item in capabilities"
        v-bind:key="item.id"
        v-bind:product="id"
        v-bind:capability="item.id"
        v-bind:systems="item.systems" />
    </div>
  </div>
</template>

<script>
  import ProductCapability from './ProductCapability';
  export default {
    name: 'Product',
    components: {
      ProductCapability
    },
    props: {
      title: {
        type: String,
        required: true
      },
      capabilities: {
        type: Array,
        required: true
      },
      id: {
        type: String,
        required: true
      }
    },
    data() {
      return {
        container_id: 'container-' + this.id
      };
    }
  };
</script>

<style scoped>


.product {
  border: 2px solid #000000;
  margin: 5px;
  position: absolute;
  border-radius: 5px;
  opacity: 0.9;
  background-color: #0b74de;
}

.product-container{
  width: 100%;
  display: flex;
  flex-direction: column;
}

.product-title{
  margin-left: 5px;
  text-align: center;
  vertical-align: middle;
  font-size: small;
  font-weight: bold;
  max-width: 150px;
}



</style>
