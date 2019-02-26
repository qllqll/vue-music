<template>
    <transition name="slide">
        <music-list :rank="rank" :title="title" :bg-image="bgImage" :songs="songs"></music-list>
    </transition>
</template>

<script type="text/ecmascript-6">
    import MusicList from 'components/music-list/music-list'
    import {
        mapGetters
    } from 'vuex'
    import {
        getMusicList
    } from 'api/rank'
    import {
        createSong
    } from 'common/js/song'
    import {
        getMusic
    } from 'api/singer'
    import {
        ERR_OK
    } from 'api/config'
    export default {
        data() {
            return {
                songs: [],
                rank: true
            }
        },
        computed: {
            title() {
                return this.topList.topTitle
            },
            bgImage() {
                return this.songs.length ? this.songs[0].image : ''
            },
            ...mapGetters([
                'topList'
            ])
        },
        components: {
            MusicList
        },
        created() {
            this._getMusicList()
        },
        methods: {
            _getMusicList() {
                if (!this.topList.id) {
                    this.$router.push('/rank')
                    return
                }
                getMusicList(this.topList.id).then((res) => {
                    if (res.code === ERR_OK) {
                        this.songs = this._normalizeSongs(res.songlist)
                    }
                }).catch((err) => {
                    console.log(err)
                })
            },
            _normalizeSongs(list) {
                let ret = []
                list.forEach((item) => {
                    const musicData = item.data
                    if (musicData.songid && musicData.albumid) {
                        getMusic(musicData.songmid).then((res) => {
                            console.log(res)
                            if (res.code === ERR_OK) {
                                const svkey = res.data.items
                                const songVkey = svkey[0].vkey
                                const newSong = createSong(musicData, songVkey)
                                ret.push(newSong)
                            }
                        })
                    }
                })
                return ret
            }
        }
    }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
.slide-enter-active, .slide-leave-active
    transition all 0.3s ease 
.slide-enter, .slide-leave-to
    transform translate3d(100%, 0, 0)
</style>
