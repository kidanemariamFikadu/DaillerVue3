<template>
  <div>
    <div class="dialed-numbers">
      {{ formattedNumber }}
    </div>
    <div class="call-duration" v-if="onCall">
      {{ minutes }}:{{ formattedSeconds }}
    </div>
    <div class="dialpad" v-if="!onCall">
      <button
        v-for="number in numbers"
        :key="number"
        @click="dial(number)"
        v-html="number === '*' ? number : number === '#' ? number : number"
      ></button>
    </div>
    <div class="call-controls" v-if="onCall">
      <button @click="mute"><i class="fas fa-microphone-slash"></i>Mute</button>
      <button @click="hold"><i class="fas fa-pause"></i>Hold</button>
      <button @click="transfer">
        <i class="fas fa-exchange-alt"></i>Transfer
      </button>
      <button @click="hangup"><i class="fas fa-phone-slash"></i>Hangup</button>
    </div>
    <div class="call-controls" v-else>
      <button @click="call"><i class="fas fa-phone"></i> CALL</button>
    </div>
  </div>
</template>


<script>
import { parsePhoneNumberFromString } from "libphonenumber-js";
export default {
  data() {
    return {
      numbers: [1, 2, 3, 4, 5, 6, 7, 8, 9, "*", 0, "#"],
      dialedNumber: "",
      onCall: false,
      isMuted: false,
      onHold: false,
      second: 0,
      intervalId: null,
    };
  },
  computed: {
    formattedNumber() {
      let phoneNumber = parsePhoneNumberFromString(this.dialedNumber, "US");
      if (phoneNumber) {
        return phoneNumber.formatNational();
      } else {
        return this.dialedNumber;
      }
    },
    minutes() {
      return Math.floor(this.second / 60);
    },
    formattedSeconds() {
      return ("0" + (this.second % 60)).slice(-2);
    },
  },
  methods: {
    dial(number) {
      this.dialedNumber += number;
    },
    call() {
      this.onCall = true;
      this.startCall();
    },
    hangup() {
      this.onCall = false;
      this.dialedNumber = "";
      clearInterval(this.intervalId);
      this.second = 0;
    },
    mute() {
      this.isMuted = !this.isMuted;
    },
    hold() {
      this.onHold = !this.onHold;
    },
    transfer() {
      // handle transfer logic here
    },
    startCall() {
      this.intervalId = setInterval(() => {
        this.second++;
      }, 1000);
    },
  },
};
</script>

<style>
.dialpad {
  display: flex;
  flex-wrap: wrap;
  width: 300px;
  margin: 0 auto;
}

.dialpad button {
  width: 33.33%;
  height: 50px;
  font-size: 24px;
  background-color: #ddd;
  border: none;
  outline: none;
  cursor: pointer;
}

.call-controls {
  display: flex;
  justify-content: space-between;
  width: 300px;
  margin: 10px auto;
}

.call-controls button {
  width: 30%;
  height: 50px;
  font-size: 18px;
  background-color: #ddd;
  border: none;
  outline: none;
  cursor: pointer;
}

.dialed-numbers {
  margin: 10px;
  font-size: 18px;
  text-align: center;
}
</style>

