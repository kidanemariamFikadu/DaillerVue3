<template>
  <div>
    <Dialpad @add-to-history="addToHistory" />
    <CallHistory :call-history="callHistory" />
  </div>
</template>


<script>
import Dialpad from "./Dialpad.vue";
import CallHistory from "./CallHistory.vue";

export default {
  components: {
    Dialpad,
    CallHistory,
  },
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
