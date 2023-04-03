<template>
  <div v-bind:id="id" class="product-capability">
    <ul v-for="c2 in systems" v-bind:key="c2.title">
      <li class="product-system">{{ c2.title }}</li>
    </ul>
  </div>
</template>

<script>
  export default {
    name: 'ProductCapability',
    props: {
      systems: {
        type: Array,
        required: true
      },
      product: {
        type: String,
        required: true
      },
      capability: {
        type: String,
        required: true
      }
    },
    data(){
      return {
        id: this.product + '-' + this.capability
      };
    }

  };
</script>

<style scoped>

.product-system {
  font-size: 8px;
  color: #470972;
}

.product-capability {
  position: absolute;
  border: 2px solid #000000;
  margin: 5px;
  max-width: 140px;
  min-width: 140px;
  min-height: 50px;
  background-color: white;
  opacity: 1.0;
  border-radius: 3px;
}


</style>
