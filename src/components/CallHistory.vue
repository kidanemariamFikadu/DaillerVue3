<template>
  <div>
    <h2>Call History</h2>
    <ul>
      <li v-for="call in callHistory" :key="call">{{ call }}</li>
    </ul>
  </div>
</template>


<script>
export default {
  data() {
    return {
      callHistory: [],
    };
  },
  methods: {
    addToHistory(call) {
      this.callHistory.push(call);
    },
  },
};
</script>
