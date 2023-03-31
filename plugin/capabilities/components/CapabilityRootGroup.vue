<template>
  <div class="capability-group-h">
    <div class="capability-group-title-h">
      <span>{{ title }}</span>
    </div>
    <div v-bind:id="container_id" class="capability-group-container-h">
      <component
        v-bind:is="item.type === 'group' ? 'CapabilityGroup' : 'Capability'"
        v-for="item in list"
        v-bind:id="item.id"
        v-bind:key="item.id"
        v-bind:title="item.title"
        v-bind:level="item.level"
        v-bind:list="item.list" />
    </div>
  </div>
</template>

<script>

  import Capability from './Capability.vue';
  import CapabilityGroup from './CapabilityGroup.vue';

  export default {
    name: 'CapabilityRootGroup',
    components: {
      Capability,
      CapabilityGroup
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


.capability-group-h {
  border: 2px solid #000000;
  margin: 5px;
  display: flex;
  flex-direction: row;
}

.capability-group-container-h {
  width: 100%;
  position: relative;
}

.capability-group-title-h {
  transform: rotate(180deg);
  writing-mode: vertical-lr;
  text-align: center;
  margin: 5px;
  font-size: large;
  font-weight: bold;
}
</style>
