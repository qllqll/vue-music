<template>
    <scroll class="suggest" :data="result" @scrollToEnd="searchMore" :pullup="pullup" ref="suggest" :beforeScroll="beforeScroll" @beforeScroll="listScroll">
        <ul class="suggest-list">
            <li @click="selectItem(item)" class="suggest-item" v-for="(item,index) in result" :key="index">
                <div class="icon">
                    <i :class="getIconCls(item)"></i>
                </div>
                <div class="name">
                    <p class="text" v-html="getDisplayName(item)"></p>
                </div>
            </li>
            <loading v-show="hasMore" title=""></loading>
        </ul>
        <div class="no-result-wrapper" v-show="!hasMore && !result.length">
            <no-result title="抱歉，暂无搜索结果"></no-result>
        </div>
    </scroll>
</template>

<script type="text/ecmascript-6">
    import {
        serach
    } from 'api/search'
    import {
        ERR_OK
    } from 'api/config'
    import {
        createSong
    } from 'common/js/song'
    import {
        getMusic
    } from 'api/singer'
    import NoResult from 'base/no-result/no-result'
    import Loading from 'base/loading/loading'
    import Scroll from 'base/scroll/scroll'
    import Singer from 'common/js/singer'
    import {
        mapMutations,
        mapActions
    } from 'vuex'
    const TYPE_SINGER = 'singer'
    const perpage = 20
    export default {
        props: {
            query: {
                type: String,
                default: ''
            },
            showSinger: {
                type: Boolean,
                default: true
            }
        },
        components: {
            Scroll,
            Loading,
            NoResult
        },
        data() {
            return {
                page: 1,
                result: [],
                test: [],
                pullup: true,
                hasMore: true,
                beforeScroll: true
            }
        },
        watch: {
            query() {
                this.search()
            }
        },
        methods: {
            selectItem(item) {
                if (item.type === TYPE_SINGER) {
                    const singer = new Singer({
                        id: item.singermid,
                        name: item.singername
                    })
                    this.$router.push({
                        path: `/search/${singer.id}`
                    })
                    this.setSinger(singer)
                } else {
                    this.insertSong(item)
                }
                this.$emit('select')
            },
            searchMore() {
                if (!this.hasMore) {
                    return
                }
                this.page++
                    serach(this.query, this.page, this.showSinger, perpage).then((res) => {
                        if (res.code === ERR_OK) {
                            this._checkMore(res.data)
                            this._getResult(res.data, true)
                        }
                    }).catch(() => {
                        this.hasMore = false
                    })
            },
            search() {
                this.page = 1
                this.result = []
                this.$refs.suggest.scrollTo(0, 0)
                this.hasMore = true
                serach(this.query, this.page, this.showSinger, perpage).then((result) => {
                    if (result.code === ERR_OK) {
                        this._checkMore(result.data)
                        this._getResult(result.data, false)
                    }
                }).catch(() => {
                    this.hasMore = false
                })
            },
            getIconCls(item) {
                if (item.type === TYPE_SINGER) {
                    return 'icon-mine'
                } else {
                    return 'icon-music'
                }
            },
            getDisplayName(item) {
                if (item.type === TYPE_SINGER) {
                    return item.singername
                } else {
                    return `${item.name}-${item.singer}`
                }
            },
            refresh() {
                this.$refs.suggest.refresh()
            },
            listScroll() {
                this.$emit('listScroll')
            },
            ...mapMutations({
                setSinger: 'SET_SINGER'
            }),
            _getResult(data, ispullup) {
                if (data.zhida && data.zhida.singerid) {
                    this.result.push({ ...data.zhida,
                        ...{
                            type: TYPE_SINGER
                        }
                    })
                }
                if (data.song) {
                    this._normalizeSongs(data.song.list, ispullup)
                }
            },
            _normalizeSongs(list, ispullup) {
                list.forEach((musicData, index) => {
                    if (musicData.songmid) {
                        getMusic(musicData.songmid).then((res) => {
                            if (res.code === ERR_OK) {
                                const svkey = res.data.items
                                const songVkey = svkey[0].vkey
                                const newSong = createSong(musicData, songVkey)
                                this.result.push(newSong)
                            }
                        })
                    }
                })
            },
            _checkMore(data) {
                const song = data.song
                if (!song.list.length || song.curpage * perpage >= song.totalnum) {
                    this.hasMore = false
                }
            },
            ...mapActions([
                'insertSong'
            ])
        }
    }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
@import "~common/stylus/variable"
@import "~common/stylus/mixin"

.suggest
    height 100%
    overflow hidden
    .suggest-list
        padding 0 30px
        .suggest-item
            display flex
            align-items center
            padding-bottom 20px
        .icon
            flex 0 0 30px
            width 30px
            [class^='icon-']
                font-size 14px
                color $color-text-d
        .name
            flex 1
            font-size $font-size-medium
            color $color-text-d
            overflow hidden
            .text
                no-wrap() 
    .no-result-wrapper
        position absolute
        width 100%
        top 50%
        transform translateY(-50%)

</style>
