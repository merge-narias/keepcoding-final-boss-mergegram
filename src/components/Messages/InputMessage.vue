<template>
  <div class="sendMessage">
    <!-- <input type="text" v-model="text" @keypress.enter="sendMessage" width="100%"
            placeholder="Escribe tu mensaje..."> -->
    <discord-picker
      input
      :value="text"
      @update:value="text = $event"
      @emoji="setEmoji"
      @keyup.enter="sendMessage"
    />
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";
import DiscordPicker from "vue3-discordpicker";
import { useStore } from "vuex";
import { db, firebase } from "@/firebase";

export default defineComponent({
  name: "InputMessage",
  components: {
    DiscordPicker,
  },
  setup() {
    const text = ref<string>("");
    const store = useStore();
    const cMessage = db.collection("messages");

    const setEmoji = (emoji: any): void => {
      // console.log(emoji);
    };
    cMessage.onSnapshot((snapShot) => {
      snapShot.docChanges().forEach((change) => {
        if (change.type === "added") {
          store.dispatch("getMessages");
        }
      });
    });

    const sendMessage = async (): Promise<void> => {
      const tmp = text.value;
      text.value = "";
      cMessage.add({
        type: "text",
        text: tmp,
        date: firebase.firestore.FieldValue.serverTimestamp(),
        senderId: store.state.user.uid,
        receiverId: store.state.recipientUid,
      });
    };

    return {
      text,
      setEmoji,
      sendMessage,
    };
  },
});
</script>

<style>
.sendMessage {
  display: flex;
  align-items: flex-end;
}

.emojibutton__active .vue3-discord-emojipicker__emojibutton {
  width: 22px;
}
</style>
