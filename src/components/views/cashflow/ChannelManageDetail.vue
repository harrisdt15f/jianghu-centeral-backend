<template>
    <div class="cont" ref="channelManageDetail">
        <div class="filter p10">
            <ul class="left">
                 <li>
                    <span>通道名称</span>
                    <Select v-model="filter.channel" :options="channel_opt"></Select>
                </li>
                <li>
                    <span>厂商名称</span>
                    <Select v-model="filter.vendor" :options="vendor_opt"></Select>
                </li>
                <li>
                    <span>分类名称</span>
                    <Select v-model="filter.sort" :options="sort_opt" ></Select>
                </li>
                <li>
                    <span>日期选择</span>
                    <!-- <Date v-model="filter.dates[0]" />
                    <span style="margin:0 5px;">~</span>
                    <Date v-model="filter.dates[1]" /> -->
                    <Date type="daterange" v-model="filter.dates" />
                </li>
                <li>
                    <button class="btn-blue">查询</button>
                </li>
            </ul>
        </div>
        <div>
                <ul class="opera-list">
                    <li v-for="(item, index) in list" :key="index">
                        <span style="min-width:150px;text-align:right;">{{timeAgo(item.created_at)}}</span>
                        <div class="pic-cont">
                            <img
                                class="img"
                                src="../../../assets/image/game/img (1).jpg"
                                alt="图片丢失"
                            />
                            <div :class="[index!==list.length-1?'vertical-bar':'']"></div>
                        </div>
                        <div class="opera-cont">
                            <div class="cont-left">
                                <div class="cont-title">{{item.title}}</div>
                                <!-- <div class="mt8">
                                    <span>操作时间:</span>
                                    <span>{{item.created_at}}</span>
                                </div> -->
                            </div>
                            <div class="cont-right">
                                <!-- <button class="btn-blue" @click="detail(item)">查看详情</button> -->
                                <ul>
                                    <li class="detail">
                                        <div class="item-detail">
                                            <span style="color:#4c8bfd">详情</span>
                                        </div>
                                        <div>
                                            <span>管理员</span>
                                            <span>: </span>
                                            <span>{{item.admin_name}}</span>
                                        </div>
                                        <div>
                                            <span>时间</span>
                                            <span>: </span>
                                            <span>{{item.created_at}}</span>
                                        </div>
                                        <div>
                                            <span>来源</span>
                                            <span>: </span>
                                            <span>{{item.origin}}</span>
                                        </div>
                                        <div>
                                            <span>IP</span>
                                            <span>: </span>
                                            <span>{{item.ip}}</span>
                                        </div>
                                        <div>
                                            <span>浏览器</span>
                                            <span>: </span>
                                            <img
                                                class="explorer-img"
                                                :src="getExplorerSrc(item.user_agent)"
                                                alt="图片加载失败"
                                            />
                                            <span>{{item.user_agent}}</span>
                                        </div>
                                    </li>
                                </ul>
                            </div>
                        </div>
                    </li>
                </ul>
            </div>
        <!-- <Dialog :show.sync="dia_show" title="操作详情">
            <div class="dia-inner">
                <ul class="dia-cont">
                    <li>
                        <div class="bold-blue">操作设置:</div>
                        <div class="mt8">修改抢红包金额</div>
                        <div class="mt8">修改抢红包VIP7特权倍数</div>
                    </li>
                    <li class="mt30">
                        <div class="bold-blue mt8">删除内容:</div>
                        <div class="mt8">删除14:30抢红包活动</div>
                    </li>
                </ul>
            </div>
        </Dialog> -->
    </div>
</template> <script>
export default {
    data() {
        return {
             filter: {
                vendor: '',
                name: '',
                sort: '',
                dates: []

            },
            channel_opt: [
                { label: '全部', value: '' },
                { label: '红牛', value: '1' },
                { label: '东鹏？', value: '2' }
            ],
            vendor_opt: [
                { label: '全部', value: '0' },
                { label: '抢庄牛牛', value: '1' },
                { label: '百家乐', value: '2' }
            ],
            sort_opt: [
                {
                    label: '抢庄牛牛',
                    value: '1'
                },
                {
                    label: '百家乐',
                    value: '2'
                }
            ],
            list: [],
            dia_show: false
        }
    },
        methods: {
        detail(item) {
            this.dia_show = true
            this.curr_row = item
        },
        // 第一次加载
        firstLoad() {
            this.getList().then(res => {
                if (res.data) {
                    this.list = res.data.data
                    this.total = res.data.total
                }
            })
        },
        getList() {
            return new Promise((resolve, reject) => {
                let created_at = ''
                if (this.filter.created_at[0] && this.filter.created_at[1]) {
                    created_at = JSON.stringify(this.filter.created_at)
                }
                let para = {
                    data_id: this.id, // Id
                    admin_name: this.filter.admin_name, // 管理员名称
                    created_at: created_at, // 管理员时间
                    pageSize: this.pageSize,
                    page: this.pageNo
                }

                let params = window.all.tool.rmEmpty(para)
                let { url, method } = this.$api.game_edit_detail_list
                this.$http({ method, url, params }).then(res => {
                    // console.log('列表👌👌👌👌: ', res)
                    if (res && res.code === '200') {
                        resolve(res)
                    } else {
                        // reject(res)
                    }
                })
            })
        },

        timeAgo(time) {
            let reg = /^(\d{1,4})(-|\/)(\d{1,2})\2(\d{1,2}) (\d{1,2}):(\d{1,2}):(\d{1,2})$/
            if (!reg.test(time)) return
            let dateTimeStamp = new Date(time)

            //dateTimeStamp是一个时间毫秒，注意时间戳是秒的形式，在这个毫秒的基础上除以1000，就是十位数的时间戳。13位数的都是时间毫秒。
            var minute = 1000 * 60 //把 分，时，天，周，半个月，一个月用毫秒表示
            var hour = minute * 60
            var day = hour * 24
            var week = day * 7
            var halfamonth = day * 15
            var month = day * 30
            var now = new Date().getTime() //获取当前时间毫秒
            var diffValue = now - dateTimeStamp //时间差

            if (diffValue < 0) {
                return
            }
            var minC = diffValue / minute //计算时间差的分，时，天，周，月
            var hourC = diffValue / hour
            var dayC = diffValue / day
            var weekC = diffValue / week
            var monthC = diffValue / month
            let result = '--'
            if (monthC >= 1 && monthC <= 3) {
                result = ' ' + parseInt(monthC) + '月前'
            } else if (weekC >= 1 && weekC <= 3) {
                result = ' ' + parseInt(weekC) + '周前'
            } else if (dayC >= 1 && dayC <= 6) {
                result = ' ' + parseInt(dayC) + '天前'
            } else if (hourC >= 1 && hourC <= 23) {
                result = ' ' + parseInt(hourC) + '小时前'
            } else if (minC >= 1 && minC <= 59) {
                result = ' ' + parseInt(minC) + '分钟前'
            } else if (diffValue >= 0 && diffValue <= minute) {
                result = '刚刚'
            } else {
                var datetime = new Date()
                datetime.setTime(dateTimeStamp)
                var Nyear = datetime.getFullYear()
                var Nmonth =
                    datetime.getMonth() + 1 < 10
                        ? '0' + (datetime.getMonth() + 1)
                        : datetime.getMonth() + 1
                var Ndate =
                    datetime.getDate() < 10
                        ? '0' + datetime.getDate()
                        : datetime.getDate()
                var Nhour =
                    datetime.getHours() < 10
                        ? '0' + datetime.getHours()
                        : datetime.getHours()
                var Nminute =
                    datetime.getMinutes() < 10
                        ? '0' + datetime.getMinutes()
                        : datetime.getMinutes()
                var Nsecond =
                    datetime.getSeconds() < 10
                        ? '0' + datetime.getSeconds()
                        : datetime.getSeconds()
                result = Nyear + '-' + Nmonth + '-' + Ndate
            }
            return result
        },

        // 滚动加载
        scroll() {
            let isLoading = false
            let ele = this.$refs.channelManageDetail
            let self = this
            ele.onscroll = () => {
                // 距离底部200px时加载一次
                let scrollHeight = ele.scrollHeight
                let scrollTop = ele.scrollTop
                let offsetHeight = ele.offsetHeight
                let bottomOfWindow = scrollHeight - scrollTop - offsetHeight
                if (bottomOfWindow < 200 && isLoading == false) {
                    let totalPage = Math.ceil(this.total / this.pageSize)

                    // 如果是加载 到最后一页
                    if (this.pageNo > totalPage) return
                    isLoading = true
                    this.pageNo++ // 请求下一页
                    this.getList().then(res => {
                        isLoading = false
                        if (res.data) {
                            this.list = this.list.concat(res.data.data || [])
                        }
                    })
                }
            }
        },
        getExplorerSrc(explorer_name) {
            let name = window.all.tool.getExploreName(explorer_name)

            switch (name) {
                case 'Opera':
                    return require('../../../assets/image/browser/opera.png')
                    break
                case 'IE':
                    return require('../../../assets/image/browser/IE.png')
                    break
                case 'Edge':
                    return require('../../../assets/image/browser/IE.png')
                    break
                case 'Firefox':
                    return require('../../../assets/image/browser/firefox.png')
                case 'Safari':
                    return require('../../../assets/image/browser/safari.png')
                    break
                case 'Chrome':
                    return require('../../../assets/image/browser/chrome.png')
                    break
                case 'IE>=11':
                    return require('../../../assets/image/browser/IE.png')
                    break

                default:
                    // return require('../../../assets/image/browser/IE.png')
                    break
            }
        }
    },
    // watch: {
    //     select(val) {
    //         this.initOpt()
    //     }
    // },
    mounted() {
        // this.initOpt()
        // this.getExplorerSrc()
        // this.firstLoad()
        // this.scroll()
    }
}
</script>

<style scoped>
.cont {
    width: 1000px;
    min-height: 500px;
    max-height: 80vh;
    overflow: auto;
}

.cont-left {
    width: 200px;
    margin-left: 10px;
}

.opera-list {
    /* margin-left: 100px; */
    width: 1700px;
    margin: 20px auto 0 auto;
    /* border: 1px solid #000; */
}
.opera-list > li {
    display: flex;
    align-items: center;
    padding-bottom: 20px;
    /* overflow: hidden; */
}
.opera-list > li .pic-cont {
    position: relative;
    width: 40px;
    height: 40px;
    margin-left: 10px;
}
.opera-list > li .img {
    position: relative;
    width: 40px;
    height: 40px;
    border-radius: 50%;
    z-index: 1;
}
/* 竖线 */
.opera-list > li .vertical-bar {
    position: absolute;
    top: 40px;
    left: 50%;
    bottom: 0;
    width: 2px;
    height: 180px;
    background: #eee;
}

.opera-list > li .opera-cont {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 1300px;
    /* height: 60px; */
    padding: 10px;
    margin-left: 10px;
    background: #f2f2f2;
    border-radius: 5px;
}
.opera-list > li .cont-title {
    font-weight: bold;
    color: #4c8bfd;
}
.detail .item-detail {
    font-weight: bold;
    /* color: #4c8bfd; */
}
.detail div > span:first-child {
    display: inline-block;
    min-width: 5em;
    margin-top: 5px;
    margin-right: 10px;
    text-align-last: justify; /* ie9*/
    font-size: 1.1em;
    color: #444;
}

.detail img {
    width: 20px;
    height: 20px;
}
.mt8 {
    margin-top: 8px;
}
.mt30 {
    margin-top: 30px;
}
.dia-inner {
    display: flex;
    justify-content: center;
}
.explorer-img {
    max-width: 25px;
    max-height: 25px;
}
</style>