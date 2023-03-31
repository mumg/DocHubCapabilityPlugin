<template>
  <div class="capability" v-bind:class="cls">
    <div class="capability-title">
      <span>{{ title }}</span>
    </div>
    <ul v-for="c2 in list" v-bind:key="c2.title">
      <li class="capability-system">{{ c2.title }}</li>
    </ul>
  </div>
</template>

<script>
  export default {
    name: 'Capability',
    props: {
      title: {
        type: String,
        required: true
      },
      level: {
        type: String,
        required: true
      },
      list: {
        type: Array,
        required: true
      }
    },
    data() {
      return {
        cls: 'capability-' + this.level
      };
    }
  };
</script>

<style scoped>

.capability-system {
  font-size: 8px;
}

.capability {
  position: absolute;
  border: 2px solid #000000;
  margin: 5px;
  max-width: 140px;
  min-width: 140px;
  min-height: 50px;
}

.capability-title {
  font-size: 10px;
  text-align: center;
}

.capability-none{
  background-color: #F8CECC;
}

.capability-low{
  background-color: #FFF2CC;
}

.capability-moderate{
  background-color: #D5E8D4;
}

.capability-high {
  background-color: #CDEB8B;
}
</style>
