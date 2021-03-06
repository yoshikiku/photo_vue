<template>
    <div class="photo-list">
        <div class="grid">
            <Photo
                class="grid__item"
                v-for="photo in photos"
                :key="photo.id"
                :item="photo"
                @like="onLikeClick" />
        </div>
        <Pagination :current-page="currentPage" :last-page="lastPage" />
    </div>
</template>

<script>
    import { OK } from '../util'
    import Photo from '../components/Photo.vue'
    import Pagination from '../components/Pagination.vue'

    export default {
        components: {
            Photo,
            Pagination
        },
        props: {
            page: {
                type: Number,
                required: false,
                default: 1
            }
        },
        // <Pagination> コンポーネントに渡すため、
        // 現在ページ（currentPage）と総ページ数（total）を data に追加します。
        data() {
            return {
                photos: [],
                currentPage: 0,
                lastPage: 0
            }
        },
        methods: {
            async fetchPhotos() {
                // 総ページ数と現在ページは API のレスポンスに含まれています。
                // そこで fetchPhotos メソッドの最後に、
                // 追加した data 変数にレスポンスの該当する値を代入する記述を追加します。
                const response = await axios.get(`/api/photos/?page=${this.page}`)

                if (response.status !== OK) {
                    this.$store.commit('error/setCode', response.status)
                    return false
                }

                this.photos = response.data.data
                this.currentPage = response.data.current_page
                this.lastPage = response.data.last_page
            },
            onLikeClick({ id, liked }) {
                if (!this.$store.getters['auth/check']) {
                    alert('いいね機能を使うにはログインしてください。')
                    return false
                }

                if (liked) {
                    this.unlike(id)
                } else {
                    this.like(id)
                }
            },
            async like(id) {
                const response = await axios.put(`/api/photos/${id}/like`)

                if (response.status !== OK) {
                    this.$store.commit('error/setCode', response.status)
                    return false
                }

                this.photos = this.photos.map(photo => {
                    if (photo.id === response.data.photo_id) {
                        photo.likes_count += 1
                        photo.liked_by_user = true
                    }
                    return photo
                })
            },
            async unlike(id) {
                const response = await axios.delete(`/api/photos/${id}/like`)

                if (response.status !== OK) {
                    this.$store.commit('error/setCode', response.status)
                    return false
                }

                this.photos = this.photos.map(photo => {
                    if (photo.id === response.data.photo_id) {
                        photo.likes_count -= 1
                        photo.liked_by_user = false
                    }
                    return photo
                })
            }
        },
        // $route を監視してページが切り替わったときに fetchPhotos が実行
        watch: {
            $route: {
                async handler() {
                    await this.fetchPhotos()
                },
                // trueでコンポーネントが生成されたタイミングでも実行
                immediate: true
            }
        }
    }
</script>
