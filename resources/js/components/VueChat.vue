<template>
    <div class="chat">
        <div class="columns main-wrapper">

            <vue-chat-channels :channels="channels" :active-channel="activeChannel" @channelChanged="onChannelChanged"></vue-chat-channels>

            <vue-chat-messages :messages="messages"></vue-chat-messages>

            <vue-chat-participants :participants="participants"></vue-chat-participants>

        </div>

        <vue-chat-new-message :active-channel="activeChannel" :username="username"></vue-chat-new-message>
    </div>
</template>

<script>
    export default {
        props: ['channels'],

        data() {
            return {
                activeChannel: this.channels[0].channel_id,
                messages: [],
                username: document.getElementById("chat_name").value,
                participants: [],
            };
        },

        methods: {
            fetchMessages() {
                let endpoint = `/channels/${this.activeChannel}/messages`;

                axios.get(endpoint)
                    .then(({ data }) => {
                        this.messages = data;
                    });
            },
            onChannelChanged(id) {
                this.activeChannel = id;

                this.fetchMessages();
            }
        },

        created() {
            this.fetchMessages();

            this.socket = io(`https://app.fittut.com:3002?username=${this.username}`);

            for (let channel of this.channels) {
                this.socket.on(`${channel.name}:App\\Events\\MessageSent`, data => {
                    this.messages.push(data.data);
                });
            }

            // Push a new "virtual" message to the messages array after someone has
            // entered the chat. "virtual" means this message won't be persisted in the
            // database and will be only shown once
            this.socket.on(`user-joined`, data => {
                this.participants = data.participants;

                this.messages.push({
                    message: `${data.username} sohbete katıldı.`,
                    author_username: 'system',
                });
            });

            // Same thing for after someone has disconnected from the chat
            this.socket.on(`user-left`, data => {
                this.participants = data.participants;

                this.messages.push({
                    message: `${data.username} sohbetten ayrıldı.`,
                    author_username: 'system',
                });
            });
        },
    }
</script>

<style>

</style>