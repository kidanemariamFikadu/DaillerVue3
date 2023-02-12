<template>
  <div id="phone">
    <div v-if="showView" class="view">
      <button class="close-button" @click="showView = !showView">X</button>
      <div class="tabs">
        <div
          class="tab"
          :class="{ active: selectedTab === 'dialpad' }"
          @click="selectedTab = 'dialpad'"
        >
          <span class="material-symbols-outlined">dialpad</span>
        </div>
        <div
          class="tab"
          :class="{ active: selectedTab === 'history' }"
          @click="selectedTab = 'history'"
        >
          <span class="material-symbols-outlined">history</span>
        </div>
      </div>
      <div v-if="selectedTab === 'dialpad'">
        <div class="dialed-numbers" @paste="onNumberPaste">
          {{ dialedNumber }}
        </div>
        <div
          :style="{ color: callDurationColor }"
          class="call-duration dialed-numbers"
          v-if="onCall"
        >
          {{ minutes }}:{{ formattedSeconds }}
        </div>
        <div v-if="errorMessage" class="error-message">
          {{ errorMessage }}
        </div>
        <div class="dialpad" v-if="!onCall">
          <div class="keypad">
            <button
              ref="specialButton"
              class="keypad-button"
              v-for="number in numbers"
              :key="number"
              @click="numberClicked(number)"
              v-html="
                number === '*' ? number : number === '#' ? number : number
              "
            ></button>
          </div>
        </div>
        <div class="controls">
          <div v-if="!onCall">
            <button @click="call()" class="material-symbols-outlined">
              Call
            </button>
            <button @click="clearLastDigit" class="material-symbols-outlined">
              backspace
            </button>
          </div>
          <div v-if="onCall">
            <button class="material-symbols-outlined">mic_off</button>
            <button class="material-symbols-outlined">phone_paused</button>
            <button class="material-symbols-outlined">send_to_mobile</button>
            <button class="material-symbols-outlined">add_call</button>
            <button @click="hangup()" class="material-symbols-outlined danger">
              call_end
            </button>
          </div>
        </div>
      </div>
      <div v-if="selectedTab === 'history'">
        <div>
          <table class="table" v-if="showTable == true">
            <tbody>
              <tr
                v-for="call in callHistory"
                :key="call.id"
                @click="callOut(call.phone)"
              >
                <td>
                  <span
                    v-if="call.direction === 'outgoing'"
                    class="material-symbols-outlined outgoing-call"
                    >call_made</span
                  >
                  <span
                    v-if="
                      call.direction === 'incoming' && call.status !== 'missed'
                    "
                    class="material-symbols-outlined incoming-call"
                    >call_received</span
                  >
                  <span
                    v-if="call.status === 'missed'"
                    class="material-symbols-outlined missed-call"
                    >call_missed_outgoing</span
                  >
                </td>
                <td>{{ call.phone }}</td>
                <td>{{ call.duration }}</td>
                <td>{{ timeDifference(call.startTime) }}</td>
              </tr>
            </tbody>
          </table>
          <div v-else>No Data To Display</div>
        </div>
      </div>
    </div>
    <div class="floating-button" @click="showView = !showView">
      <div class="circle material-symbols-outlined">call</div>
    </div>
  </div>
</template>

<script>
import moment from "moment";

export default {
  data() {
    return {
      numbers: [1, 2, 3, 4, 5, 6, 7, 8, 9, "*", 0, "#"],
      showView: false,
      selectedTab: "dialpad",
      onCall: false,
      dialedNumber: "",
      showTable: true,
      second: 0,
      callDurationColor: "black",
      errorMessage: "",
      specialButton: "*",
      callHistory: [
        {
          id: 1,
          direction: "outgoing",
          phone: "+254796671443",
          startTime: "2023-02-01T12:30:00.000Z",
          endTime: "2022-12-01T12:30:00.000Z",
          duration: "30",
          status: "completed",
        },
        {
          id: 2,
          direction: "incoming",
          phone: "+14155552222",
          startTime: "2023-02-10T09:15:00.000Z",
          endTime: "2022-12-02T09:15:00.000Z",
          duration: "15",
          status: "missed",
        },
        {
          id: 3,
          direction: "outgoing",
          phone: "+14155551111",
          startTime: "2022-12-02T09:15:00.000Z",
          endTime: "2022-12-03T14:45:00.000Z",
          duration: "45",
          status: "completed",
        },
        {
          id: 1,
          direction: "incoming",
          phone: "+14155553333",
          startTime: "2022-12-01T12:30:00.000Z",
          endTime: "2022-12-01T12:30:00.000Z",
          duration: "30",
          status: "completed",
        },
        {
          id: 2,
          direction: "incoming",
          phone: "+14155552222",
          startTime: "2022-12-02T09:15:00.000Z",
          endTime: "2022-12-02T09:15:00.000Z",
          duration: "15",
          status: "completed",
        },
        {
          id: 3,
          direction: "incoming",
          phone: "+14155551111",
          startTime: "2022-12-03T14:45:00.000Z",
          endTime: "2022-12-03T14:45:00.000Z",
          duration: "45",
          status: "completed",
        },
      ],
    };
  },
  mounted() {
    const view = document.querySelector("#phone");
    view.addEventListener("keydown", (event) => {
      if (event.key >= 0 && event.key <= 9) {
        if (this.dialedNumber.length < 13) {
          this.errorMessage = "";
          this.dialedNumber += event.key.toString();
        }
      } else if (event.key === "Backspace" || event.key === "Delete") {
        this.clearLastDigit();
      }
    });
  },
  computed: {
    minutes() {
      return Math.floor(this.second / 60);
    },
    formattedSeconds() {
      return ("0" + (this.second % 60)).slice(-2);
    },
    formattedSpecialButton() {
      return this.dialedNumber === "+" ? "+" : this.specialButton;
    },
  },
  methods: {
    numberClicked(number) {
      this.errorMessage = "";
      if (number == "*" && this.dialedNumber.length === 0) {
        this.dialedNumber = "*";
        this.$refs.specialButton.textContent = "+";
      } else if (this.dialedNumber == "*") {
        if (number === "*") {
          this.dialedNumber = "+";
        } else {
          this.dialedNumber += number.toString();
        }
      } else if (number != "*") {
        if (this.dialedNumber.length < 13) {
          this.dialedNumber += number.toString();
        }
      }
    },
    clearLastDigit() {
      this.dialedNumber = this.dialedNumber.slice(0, -1);
      this.errorMessage = "";
    },
    timeDifference(data) {
      var moment_date = moment(data).format("YYYY-MM-DD, h:mm:ss a");
      return moment(moment_date)
        .startOf("minute")
        .fromNow();
    },
    call() {
      if (!this.dialedNumber) {
        this.errorMessage = "please enter phone number";
      } else {
        const phoneNumberRegex9 = /^\d{9}$/;
        const phoneNumberRegex10 = /^\d{10}$/;
        const phoneNumberRegex12 = /^\d{12}$/;
        const phoneNumberRegex13 = /^\+?[1-9]\d{9,11}$/;
        if (
          phoneNumberRegex9.test(this.dialedNumber) ||
          phoneNumberRegex10.test(this.dialedNumber) ||
          phoneNumberRegex12.test(this.dialedNumber) ||
          phoneNumberRegex13.test(this.dialedNumber)
        ) {
          this.onCall = true;
          this.startCall();
        } else {
          this.errorMessage = "please enter correct number";
        }
      }
    },
    startCall() {
      this.intervalId = setInterval(() => {
        this.second++;
      }, 1000);
    },
    hangup() {
      this.callDurationColor = "red";
      this.errorMessage = "call ended";
      clearInterval(this.intervalId);
      setTimeout(() => {
        setTimeout(() => {
          this.callDurationColor = "black";
          this.errorMessage = "";
          this.onCall = false;
          this.second = 0;
          this.dialedNumber = "";
        }, 2000);
      }, 0);
    },
    callOut(phone) {
      this.dialedNumber = phone;
      this.selectedTab = "dialpad";
      this.call();
    },
    onNumberPaste(evt) {
      var p = evt.clipboardData
        .getData("text")
        .replace(/\D+/g, "")
        .replace(/(\d{1})(\d{3})(\d{3})(\d{4})/, "$1$2$3$4");
      this.dialedNumber = p;
    },
  },
};
</script>

<style>
.dialed-numbers {
  font-size: 32px;
  min-height: 33px;
  overflow: hidden;
  /* display: flex; */
  /* box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.2); */
}

.floating-button {
  position: fixed;
  bottom: 16px;
  right: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: green;
  width: 56px;
  height: 56px;
  border-radius: 28px;
  cursor: pointer;
}

.circle {
  color: #fff;
}

.view {
  position: fixed;
  bottom: 76px;
  right: 16px;
  background-color: white;
  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.2);
  border-radius: 8px;
  padding: 16px;
  width: 300px;
  z-index: 1000;
  min-height: 480px;
}

.view .close-button {
  position: absolute;
  top: -10px;
  right: -10px;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background-color: green;
  color: #fff;
  font-size: 24px;
  line-height: 28px;
  text-align: center;
  cursor: pointer;
}

.tabs {
  display: flex;
  justify-content: center;
  border-bottom: 1px solid #ddd;
  margin-bottom: 16px;
}

.tab {
  padding: 8px 16px;
  cursor: pointer;
  width: 100%;
}

.tab.active {
  border-bottom: 2px solid green;
}

.controls {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  width: 300px;
  margin: 10px;
  align-items: center;
}

.controls button {
  align-items: center;
  justify-content: center;
  border-radius: 28px;
  width: 56px;
  height: 56px;
  margin: 5px;
}

.controls button:hover {
  color: #fff;
  background-color: #000;
  cursor: pointer;
}

.danger {
  border-color: white;
  color: white;
  background-color: red;
}

.danger:hover {
  color: #fff;
  background-color: rgb(rgb(229, 42, 42));
  cursor: pointer;
}

@keyframes controls {
  0% {
    top: 0em;
  }
  40% {
    top: 0em;
  }
  43% {
    top: -0.9em;
  }
  46% {
    top: 0em;
  }
  48% {
    top: -0.4em;
  }
  50% {
    top: 0em;
  }
  100% {
    top: 0em;
  }
}

@media all and (max-width: 30em) {
  .controls {
    display: block;
    margin: 0.4em auto;
  }
}

.dialpad {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.keypad {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

.keypad-button {
  width: 60px;
  height: 60px;
  border: 1px solid lightgray;
  font-size: 24px;
  text-align: center;
  margin: 10px;
  cursor: pointer;
}

.table {
  width: 100%;
}

.table tr {
  cursor: pointer;
}

.table tr:hover {
  background: #ddd;
  border: #ddd;
}

.missed-call {
  color: red;
}

.outgoing-call {
  color: blue;
}

.incoming-call {
  color: #4f7942;
}

.error-message {
  color: red;
}
</style>
