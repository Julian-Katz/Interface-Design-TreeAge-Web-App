<template>
  <div class="container">
    <div class="header">
      <button type="button" @click="handleConnectDevice">
        Start
      </button>
      <button type="button" id="reset">
        Reset
      </button>
      <p>
        <span>{{ treeGirthM }}</span>
        m
      </p>
    </div>
    <div class="chat">
      <Message
        v-for="message in currentChat"
        :key="message"
        v-html="message.text"
        :userMessage="message.userMessage"
      >
      </Message>
    </div>
    <div class="input">
      <input v-model="textInput"/>
      <button
        @click="handleTextInput"
        type="button"
      >
        Senden
      </button>
    </div>
  </div>
</template>

<script>
import Message from './components/Message.vue';

export default {
  name: 'Tree Age',
  components: {
    Message,
  },
  data() {
    return {
      textInput: '',
      lengthCharacteristic: null,
      treeGirthMm: 0,
      currentTree: null,
      invalidInput: false,
      messages: {
        kindOfThreeNotFound: {
          text: 'Baumart nicht gefunden! Versuche es erneut. Folgende Baumarten unterstütze ich:',
          userMessage: false,
        },
      },
      currentChat: [
        {
          text: 'Wilkommen!<br>Um das Alter eines Baumes zu bestimmen benötige ich einige Informationen.Zuerst muss ich wissen um welche Baumart es sich handelt. Gebe den Namen der Baumart ein.',
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
      console.log(event.target.value.getUint32(0, true));
      this.treeGirthMm = event.target.value.getUint32(0, true);
    },
    handleTextInput() {
      this.currentChat.push(
        {
          text: this.textInput,
          userMessage: true,
        },
      );
      this.currentTree = this.trees.find((tree) => {
        return tree.name.toLowerCase() === this.textInput.toLowerCase();
      });
      if (!this.currentTree) {
        this.currentChat.push(this.messages.kindOfThreeNotFound);
        this.invalidInput = true;
      } else {
        this.invalidInput = false;

      };
    },
  },
};
</script>

<style lang="scss">
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
    border: 1px solid black;
    border-radius: 5px;
    background: rgb(255, 255, 255);
}
textarea {
    border: none;
    border-radius: 5px;
    padding: 0.5rem;
}

.container {
    width: 100vw;
    height: 100%;
    overflow: hidden;
    background: rgb(241, 241, 241);
    display: flex;
    justify-content: space-between;
    flex-direction: column;
}
.header {
    display: flex;
    padding: 1rem;
    height: 10%;
    background: rgb(235, 255, 235);
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
    column-gap: 1rem;
    padding: 1rem;
    height: 10%;
    background: rgb(235, 255, 235);
    input {
        width: 80%;
        height: 100%;
    }
    button {
        width: 20%;
    }
}
</style>
