<template>
  <v-layout column justify-center align-center>
    <v-flex xs12 sm8>
      <v-card min-width="400">
        <v-snackbar v-model="snackbar" :timeout="timeout" top>
          {{ message }}
          <v-btn color="pink" flat @click="snackbar = false">
            Close
          </v-btn>
        </v-snackbar>
        <v-card-title>
          <h1>Chat by Led</h1>
        </v-card-title>
        <v-card-text>
          <v-form ref="form" v-model="valid" lazy-validation>
            <v-text-field v-model="name" :counter="16" :rules="nameRules" label="Your name" required></v-text-field>

            <v-text-field v-model="room" :rules="roomRules" label="Enter the room" required></v-text-field>

            <v-btn :disabled="!valid" color="primary" @click="submit">Enter</v-btn>
          </v-form>
        </v-card-text>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
import { mapMutations } from "vuex";
export default {
  layout: "empty",
  head: {
    title: "Wellcome to the chat buddy"
  },
  sockets: {
    connect: function () {
      console.log("socket connected");
    }
  },
  data: () => ({
    snackbar: false,
    message: '',
    valid: true,
    name: "",
    nameRules: [
      v => !!v || "Enter name",
      v => (v && v.length <= 16) || "Name must not exceed 16 characters"
    ],
    room: "",
    roomRules: [v => !!v || "Enter room"]
  }),
  mounted() {
    const {message} = this.$route.query
    if (message === 'noUser') {
      this.message = 'Enter data'
    } else if (message === 'leftChat') {
      this.message = 'You have left the chat'
    }
    this.snackbar = !!this.message
  },
  methods: {
    ...mapMutations(["setUser"]),
    submit() {
      if (this.$refs.form.validate()) {
        const user = {
          name: this.name,
          room: this.room
        };

        this.$socket.emit("userJoined", user, data => {
          if (typeof data === "string") {
            console.error(data);
          } else {
            user.id = data.userId;
            this.setUser(user);
            this.$router.push("/chat");
          }
        });
      }
    }
  }
};
</script>
