<template>
  <div class="dropdown-outer" @click="sendMessage(emoji)" @keyup.enter="sendMessage(emoji)" tabindex="0" :aria-label="label" role="button">
    <div class="dropdown-item" tabindex="-1" :class="{ faded: !canPost }">
      <img class="emoji" :src="getToneImg" />
    </div>
  </div>
</template>

<script>
import { generateUUID } from "../../utils/index";

export default {
  props: {
    emoji: String,
    text: String,
    label: String,
  },

  computed: {
    canPost() {
      return this.$store.state.messages.filter((h) => h.owner === true).length < 1;
    },
    getToneImg() {
      return `chrome-extension://${this.$store.state.extensionID}/img/tones/${this.$store.state.tone}/${this.emoji}.gif`;
    },
    getUsername() {
      if (this.$store.state.isFullName) {
        return this.$store.getters.getUser("fullName");
      } else {
        return this.$store.getters.getUser("name");
      }
    },
  },
  methods: {
    sendMessage(emoji) {
      this.$store.dispatch("closeDropdown", "reactions");
      if (this.canPost) {
        this.$store.dispatch("addMessage", {
          messageId: generateUUID(),
          emoji: emoji,
          username: this.getUsername,
          img: this.$store.getters.getUser("avatar"),
          owner: true,
          tone: this.$store.state.tone,
        });

        this.$socket.sendObj({
          action: "MESSAGE",
          message: {
            id: this.$store.getters.getUser("meetingID"),
            emoji: emoji,
            username: this.getUsername,
            img: this.$store.getters.getUser("avatar"),
            tone: this.$store.state.tone,
          },
        });
        this.$gtag.event("click", {
          event_category: "Reactions",
          event_label: emoji,
          event_value: this.$store.state.tone,
        });
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.dropdown-outer {
  flex-basis: 33%;
  flex-shrink: 0;
  &:focus > .dropdown-item {
    background-color: rgba(2, 191, 165, 0.15);
    outline: 4px solid rgb(2, 191, 165);
  }
}

.dropdown-item {
  padding: 10px 18px;
  font-size: 15px;
  display: flex;
  align-items: center;
  position: relative;
  justify-content: center;
  border-radius: 8px;
  margin: 3px;
  &:hover {
    background-color: #00796b0d;
  }
}

.faded {
  opacity: 0.3;
  cursor: default;
  &:hover {
    background-color: white;
  }
}

.emoji {
  width: 100%;
}
</style>
