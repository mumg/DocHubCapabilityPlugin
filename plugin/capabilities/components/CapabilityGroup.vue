<template>
  <div v-bind:id="id" class="capability-group">
    <div class="capability-group-title">
      <span>{{ title }}</span>
    </div>
    <div v-bind:id="container_id" class="capability-group-container">
      <component
        v-bind:is="c1.type === 'group' ? 'CapabilityGroup' : 'Capability'"
        v-for="c1 in list"
        v-bind:id="c1.id"
        v-bind:key="c1.id"
        v-bind:title="c1.title"
        v-bind:level="c1.level"
        v-bind:list="c1.list" />
    </div>
  </div>
</template>

<script>

  import Capability from './Capability.vue';

  export default {
    name: 'CapabilityGroup',
    components: {
      Capability
    },
    props: {
      title: {
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


.capability-group-container {
  margin: 5px;
}

.capability-group-title {
  text-align: center;
  font-size: small;
  font-weight: bold;
}

.capability-group {
  border: 2px solid #000000;
  margin: 5px;
  position: absolute;
}
</style>
