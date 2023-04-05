<template>
  <div v-bind:id="id" class="product" v-bind:class="color">
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
      },
      type: {
        type: String,
        required: true
      }
    },
    data() {
      return {
        container_id: 'container-' + this.id,
        color: 'product-color-' + this.type
      };
    }
  };
</script>

<style scoped>

.product-color-normal {
  background-color: #E3D9EA;
}

.product-color-no-scope {
  background-color: #F8CECC;
}


.product {
  border: 2px solid #000000;
  margin: 5px;
  position: relative;
  border-radius: 5px;
  opacity: 0.9;
  width: 155px;
}

.product-container{
  width: 160px;
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
