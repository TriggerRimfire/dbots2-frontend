<template>
    <v-container fluid>
        <v-layout row>
            <v-flex xs12 class="text-xs-center">
                <h2 class="display-3">Logging in...</h2>
                <p v-if="status" class="headline">{{ status }}</p>
            </v-flex>
        </v-layout>
    </v-container>
</template>

<script>
export default {
    data() {
        return {
            status: null
        }
    },
    mounted() {
        if(!this.$route.query.code && !this.$route.query.error && !(document.referrer && ~document.referrer.indexOf("discordapp.com/oauth2/authorize"))) {
            this.status = "Logging in with Discord..."
            setTimeout(() => {
                window.location.href = `https://discordapp.com/oauth2/authorize?client_id=152988350679220225&redirect_uri=${window.location.origin}/login&scope=identify&response_type=code`
            }, 500)
            return
        }

        if(this.$route.query.error) {
            this.status = this.$route.query.error
            return
        }

        this.status = "Checking authorization code..."
        this.$store.dispatch("auth/login", {
            code: this.$route.query.code,
            redirectURI: window.location.origin + window.location.pathname
        }).then(() => {
            this.status = "Verifying current user..."
            return this.$store.dispatch("auth/getCurrentUser").then(() => {
                this.status = "Success!"
                if(this.$route.query.state && this.$route.query.state.startsWith("path:")) {
                    this.$router.push(this.$route.query.state.substring(5) || "/")
                } else {
                    this.$router.push("/")
                }
            })
        }).catch((err) => {
            console.log(err)
            if(err.response && err.response.data && err.response.data.error) {
                this.status = err.response.data.error
            } else {
                this.status = "Unknown authorization code error"
            }
        })
    }
}
</script>

<style scoped>
@media (min-width: 1904px) {
    .bot-lists .flex.xl6 {
        padding-right: 1rem;
    }
}
.bot-lists section {
    margin-bottom: 1rem;
}
</style>
