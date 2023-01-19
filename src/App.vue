<template>
  <div class="container">
    <div class="header">
      <h1>TreeAge</h1>
    </div>
    <div class="chat" ref="chat">
      <Message
        v-for="message in currentChat"
        :key="message"
        v-html="message.text"
        :userMessage="message.userMessage"
      >
      </Message>
    </div>
    <form
      class="input"
      v-if="!currentButtons"
      @submit.prevent="handleTextInput"
    >
      <input v-model="textInput"/>
      <button
        class="text-input-button"
        type="submit"
      >
        <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
          <g clip-path="url(#clip0_429_11051)">
            <path d="M5 12L4.39589 6.56299C4.22297 5.0067 5.82469 3.86433 7.23983 4.53465L19.1842 10.1925C20.7093 10.9149 20.7093 13.0851 19.1842 13.8075L7.23983 19.4653C5.82469 20.1357 4.22297 18.9933 4.39589 17.437L5 12ZM5 12H12" stroke="#79a879" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
          </g>
          <defs>
            <clipPath id="clip0_429_11051">
              <rect width="24" height="24" fill="white"/>
            </clipPath>
          </defs>
        </svg>
      </button>
    </form>
    <ButtonInput
      v-if="currentButtons"
      :buttons="currentButtons"
      @buttonClicked="handleButtonInput"
    />
  </div>
</template>

<script>
import Message from './components/Message.vue';
import ButtonInput from './components/ButtonInput.vue';

export default {
  name: 'Tree Age',
  components: {
    Message,
    ButtonInput,
  },
  data() {
    return {
      lengthCharacteristic: null,
      textInput: '',
      step: 'chooseTree',
      treeGirthMm: 0,
      currentTree: true,
      measuringMode: null,
      currentButtons: null,
      messageTimeOut: 300,
      messages: {
        kindOfThreeNotFound: {
          text: 'Baumart nicht gefunden! Versuche es erneut. Folgende Baumarten unterstütze ich: ',
          userMessage: false,
        },
        kindOfThreeFound: {
          text: 'Um das Alter des Baumes zu bestimmen brauche ich den Durchmesser',
          buttons:
          [
            {
              text: 'Messrad nutzen',
            },
            {
              text: 'manuell messen',
            },
          ],
          userMessage: false,
        },
        connectDevice: {
          text: 'Verbinde dich!',
          userMessage: false,
        },
        manualMode: {
          text: 'Gebe den Umfang des Baumes in Meter ein.',
          userMessage: false,
        },
        notANumber: {
          text: 'Gebe eine eine Nummer ein.',
          userMessage: false,
        },
      },
      currentChat: [
        {
          text: 'Wilkommen!<br>Um das Alter eines Baumes zu bestimmen benötige ich einige Informationen. Zuerst muss ich wissen um welche Baumart es sich handelt. Gebe den Namen der Baumart ein.',
          userMessage: false,
        },
      ],
      trees: [
        {
          name: 'Eiche',
          factor: 0.8,
        },
        {
          name: 'Linde',
          factor: 0.8,
        },
        {
          name: 'Eibe',
          factor: 0.7,
        },
        {
          name: 'Kastanie',
          factor: 0.7,
        },
      ],
    };
  },
  computed: {
    treeGirthM() {
      return this.treeGirthMm / 1000;
    },
    treeGirthCm() {
      return this.treeGirthMm / 10;
    },
  },
  methods: {
    handleConnectDevice() {
      navigator.bluetooth.requestDevice({
        filters: [{ name: 'MyESP32' }],
        optionalServices: ['c48e6067-5295-48d3-8d5c-0395f61792b1'],
      })
        .then((device) => device.gatt.connect())
        .then((server) => server.getPrimaryService('c48e6067-5295-48d3-8d5c-0395f61792b1'))
        .then((service) => service.getCharacteristic('c48e6067-5295-48d3-8d5c-0395f61792b2'))
        .then((characteristic) => {
          console.log(characteristic);
          this.lengthCharacteristic = characteristic;
          return characteristic.startNotifications();
        })
        .then(() => {
          this.lengthCharacteristic.addEventListener('characteristicvaluechanged', this.handleCharacteristicValueChanged);
        })
        .catch((error) => { console.error(error); });
    },
    async handleCharacteristicValueChanged(event) {
      this.treeGirthMm = event.target.value.getUint32(0, true);
    },
    handleTextInput() {
      if (this.textInput === '') {
        return;
      }
      this.currentChat.push(
        {
          text: this.textInput,
          userMessage: true,
        },
      );
      switch (this.step) {
        case 'chooseTree':
          this.currentTree = this.trees.find((tree) => {
            return tree.name.toLowerCase() === this.textInput.toLowerCase();
          });
          if (!this.currentTree) {
            setTimeout(() => {
              this.currentChat.push(this.messages.kindOfThreeNotFound);
            }, this.messageTimeOut);
            this.invalidInput = true;
            this.textInput = '';
          } else {
            setTimeout(() => {
              this.currentChat.push(this.messages.kindOfThreeFound);
            }, this.messageTimeOut);
            this.currentButtons = this.messages.kindOfThreeFound.buttons;
            this.textInput = '';
            this.step = 'chooseMeasuringMode';
          }
          break;
        case 'chooseMeasuringMode':
          if (this.textInput === 'Messrad nutzen') {
            this.step = 'measuringWheel';
            setTimeout(() => {
              this.currentChat.push(this.messages.connectDevice);
            }, this.messageTimeOut);
            this.handleConnectDevice();
            const message = {
              text: `Der Messwert beträgt ${this.treeGirthM}m`,
              userMessage: false,
              buttons: [
                {
                  text: 'bestätigen',
                },
                {
                  text: 'zurücksetzten',
                },
              ],
            };
            this.currentButtons = message.buttons;
            setTimeout(() => {
              this.currentChat.push(message);
            }, this.messageTimeOut * 2);
            this.step = 'measuringWheel';
          } else if (this.textInput === 'manuell messen') {
            this.step = 'manual';
            setTimeout(() => {
              this.currentChat.push(this.messages.manualMode);
            }, this.messageTimeOut);
            this.currentButtons = null;
          }
          break;
        case 'measuringWheel':
          if (this.textInput === 'bestätigen') {
            this.textInput = '';
            this.currentButtons = null;
            setTimeout(() => {
              this.currentChat.push({
                text: `Das Alter der ${this.currentTree.name} ist ${Math.trunc(this.treeGirthCm * this.currentTree.factor)} Jahre`,
                userMessage: false,
              });
              this.currentButtons = [
                {
                  text: 'Neu starten',
                },
              ];
              this.step = 'reset';
            }, this.messageTimeOut);
          } else if (this.textInput === 'zurücksetzten') {
            console.log('zurücksetzten');
            window.location.reload();
          }
          break;
        case 'manual':
          if (!Number.isNaN(this.textInput)) {
            this.treeGirthMm = Number(this.textInput) * 1000;
            setTimeout(() => {
              this.currentChat.push({
                text: `Das Alter der ${this.currentTree.name} ist ${Math.trunc(this.treeGirthCm * this.currentTree.factor)} Jahre`,
                userMessage: false,
              });
              this.currentButtons = [
                {
                  text: 'Neu starten',
                },
              ];
              this.step = 'reset';
            }, this.messageTimeOut);
          } else {
            setTimeout(() => {
              this.currentChat.push(this.messages.notANumber);
            }, this.messageTimeOut);
          }
          this.textInput = '';
          break;
        case 'reset':
          if (this.textInput === 'Neu starten') {
            window.location.reload();
          }
          break;
        default:
          break;
      }
    },
    handleButtonInput(button) {
      console.log('Button Input');
      this.textInput = button.text;
      this.handleTextInput();
      this.textInput = '';
    },
  },
  watch: {
    treeGirthM(newValue) {
      if (this.step === 'measuringWheel') {
        this.currentChat.at(-1).text = `Der Messwert beträgt ${newValue}m`;
      }
    },
    currentChat: {
      handler() {
        this.$refs.chat.scrollTop = this.$refs.chat.scrollHeight;
        setTimeout(() => {
          this.$refs.chat.scrollTop = this.$refs.chat.scrollHeight;
        }, 1100);
      },
      deep: true,
    },
  },
  mounted() {
  },
};
</script>

<style lang="scss">
$primary-color: #79a879;

*,
*::before,
*::after {
    box-sizing: border-box;
}
* {
    margin: 0;
    line-height: 1.5;
}
img,
picture,
video,
canvas,
svg {
    display: block;
    max-width: 100%;
}
input,
button,
textarea,
select {
    font: inherit;
}
html,
body {
    height: 100%;
    font-family: Arial, Helvetica, sans-serif;
}
#app {
  height: 100%;
}
body {
    line-height: 1.5;
}
button {
    border: none;
    border-radius: 5px;
    background: rgb(255, 255, 255);
}
input {
    border: none;
    border-radius: 5px;
    padding: 0.5rem;
}

.container {
    width: 100vw;
    height: 100%;
    overflow: hidden;
    background: #707070;
    display: flex;
    justify-content: space-between;
    flex-direction: column;
}
.header {
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 1rem;
    height: 10%;
    min-height: 4.5rem;
    background: $primary-color;
    h1 {
      color: white;
      font-size: 3rem;
    }
}
.chat {
    height: 80%;
    overflow-y: scroll;
    padding: 1rem 1rem;
    display: flex;
    align-items: flex-start;
    flex-direction: column;
    gap: 2rem;
}
.input {
    display: flex;
    justify-content: space-between;
    column-gap: 0.5rem;
    padding: 0.7rem;
    height: 7%;
    min-height: 3.2rem;
    background: $primary-color;
    input {
        width: 100%;
        height: 100%;
    }
    button {
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100%;
      width: 100%;
    }
    .text-input-button {
        border-radius: 50%;
        aspect-ratio: 1/1;
        width: unset;
        svg {
          height: 100%;
          transform: scale(1.2);
        }
    }
}
</style>
