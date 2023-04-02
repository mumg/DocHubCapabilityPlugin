<template>
  <div v-bind:id="id" class="product-capability-group" v-bind:class="color">
    <div v-bind:class="title_cls">
      <span>{{ title }}</span>
    </div>
    <div v-bind:id="container_id" class="product-capability-group-container">
      <product-capability-group
        v-for="item in list"
        v-bind:id="item.id"
        v-bind:key="item.id"
        v-bind:type="item.type"
        v-bind:title="item.title"
        v-bind:level="item.level"
        v-bind:depth="item.depth"
        v-bind:list="item.list" />
    </div>
  </div>
</template>

<script>
  export default {
    name: 'ProductCapabilityGroup',
    props: {
      title: {
        type: String,
        required: true
      },
      type: {
        type: String,
        required: true
      },
      list: {
        type: Array,
        required: true
      },
      id: {
        type: String,
        required: true
      },
      depth: {
        type: Number,
        required: true
      }
    },
    data() {
      return {
        container_id: 'container-' + this.id,
        title_cls: this.type === 'group' ? 'product-capability-group-title-h' : 'product-capability-group-title-v',
        color: 'product-color-' + (this.type === 'group' ? 0 : 1)
      };
    }
  };
</script>

<style scoped>


.product-capability-group {
  border: 2px solid #000000;
  margin: 5px;
  display: flex;
  flex-direction: row;
  border-radius: 5px;
}

.product-capability-group-container{
  width: 100%;
  display: flex;
  flex-direction: column;
}

.product-capability-group-title-h {
  transform: rotate(180deg);
  writing-mode: vertical-lr;
  text-align: center;
  margin: 5px;
  font-size: large;
  font-weight: bold;
  max-width: 150px;
}

.product-capability-group-title-v {
  margin-left: 5px;
  text-align: left;
  vertical-align: middle;
  font-size: small;
  font-weight: bold;
  max-width: 150px;
}
.product-color-0 {
}
.product-color-1 {
  background-color: #DAE8FC;
}



</style>
