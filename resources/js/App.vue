<template>
    <div>
        <header>
            <Navbar />
        </header>
        <main>
            <div class="container">
                <Message />
                <!--
                    Routerviewで
                    切り替わる場所の定義
                    URLが対応するHTMLを描写する
                -->
                <RouterView />
            </div>
        </main>
        <Footer />
    </div>
</template>

<script>
    import { NOT_FOUND, UNAUTHORIZED, INTERNAL_SERVER_ERROR } from './util'

    import Message from './components/Message.vue'
    import Navbar from './components/Navbar.vue'
    import Footer from './components/Footer.vue'

    export default {
        components: {
            Message,
            Navbar,
            Footer
        },
        computed: {
            errorCode() {
                return this.$store.state.error.code
            }
        },

        watch: {
            errorCode: {
                async handler(val) {
                    if (val === INTERNAL_SERVER_ERROR) {
                        this.$router.push('/500')
                    } else if (val === UNAUTHORIZED) {
                        await axios.get('/api/refresh-token')
                        this.$store.commit('auth/setUser', null)
                        this.$router.push('/login')
                    } else if (val === NOT_FOUND) {
                        this.$router.push('/not-found')
                    }
                },
                immediate: true
            },
            // ルートに変化があればsetCodeを解除？
            $route() {
                this.$store.commit('error/setCode', null)
            }
        }
    }
</script>
