<template>
    <div class="container">
        <!-- 厅主注单报表 -->
        <!-- 注单就是下注单 -->
        <QuickQuery :date="quick_query" @update="qqUpd" />

        <div class="filter p10">
            <ul class="left">
                <li>
                    <span>游戏平台</span>
                    <Input class="w100" v-model="filter.game_vendor_sign" />
                </li>
                <li>
                    <span>日期</span>
                    <Date type="daterange" v-model="filter.report_day" @update="timeUpdate" />
                </li>
                <li>
                    <button class="btn-blue" @click="getList">查询</button>
                    <button class="btn-blue" @click="exportExcel">导出Excel</button>
                    <button class="btn-blue" @click="exportAll">导出所有Excel</button>
                </li>
                <li>
                    <button class="btn-red" @click="clear">清除</button>
                </li>
            </ul>
        </div>

        <div class="table mt20">
            <Table :headers="headers" :column="list">
                <template v-slot:item="{row}">
                    <td>{{row.game_vendor_name}}</td>
                    <td>{{tofixedTwo(row.effective_bet)}}</td>
                    <td>{{tofixedTwo(row.tax)}}</td>
                    <td>{{tofixedTwo(row.bet)}}</td>
                    <td>{{tofixedTwo(row.commission)}}/{{tofixedTwo(row.rebate)}}</td>
                    <td>{{tofixedTwo(Number(row.win_money) - Number(row.bet))}}</td>
                    <td>{{row.day}}</td>
                </template>
            </Table>

            <Page
                class="table-page"
                :total="total"
                :pageNo.sync="pageNo"
                :pageSize.sync="pageSize"
                @updateNo="updateNo"
                @updateSize="updateSize"
            />
        </div>
    </div>
</template>
<script>
export default {
    name: 'ThirdGame',
    data() {
        return {
            filter: {
                game_vendor_sign: '',
                report_day: []
            },
            status_opt: [
                { label: '全部', value: '' },
                { label: '已派彩', value: '1' },
                { label: '未派彩', value: '2' }
            ],
            quick_query: [],
            /* table */
            headers: [
                '游戏平台',
                '总有效下注',
                '总游戏税收',
                '总投注额',
                '佣金/返利',
                '游戏盈亏',
                '日期'
            ],
            list: [],
            total: 0,
            pageNo: 1,
            pageSize: 25
        }
    },
    methods: {
        qqUpd(dates) {
            // 同步时间筛选值
            this.filter.report_day = dates
        },
        timeUpdate() {
            // 同步快捷查询按钮状态
            this.quick_query = this.filter.report_day
        },
        clear() {
            this.quick_query = []
            this.filter = {
                game_vendor_sign: '',
                report_day: []
            }
        },
        ExcelOut(data, file_name) {
            let header = this.headers
            let self = this
            function getdata(data) {
                return data.map(item => {
                    return [
                        item.game_vendor_name,
                        self.tofixedTwo(item.effective_bet),
                        self.tofixedTwo(item.tax),
                        self.tofixedTwo(item.bet),
                        self.tofixedTwo(item.commission) +
                            '/' +
                            self.tofixedTwo(item.rebate),
                        self.tofixedTwo(
                            Number(item.win_money) - Number(item.bet)
                        ),
                        item.day
                    ]
                })
            }
            import('../../../js/config/Export2Excel').then(excel => {
                excel.export_json_to_excel({
                    header: header, //表头 必填
                    data: getdata(data), //具体数据 必填
                    filename: file_name, //非必填
                    autoWidth: true, //非必填
                    bookType: 'xlsx' //非必填
                })
            })
        },
        exportExcel() {
            if (!this.list) {
                this.$toast('内容为空')
            }
            let chainName = window.all.tool.getChainName(this.$route.path)
            let filename = `${chainName} ${this.pageNo}`
            this.ExcelOut(this.list, filename)
        },
        async exportAll() {
            let self = this
            let pageSize = 100
            function getPageList(pageNo) {
                return new Promise((resolve, reject) => {
                    let para = {
                        // report_day: this.filter.report_day,
                        game_vendor_sign: self.filter.game_vendor_sign,
                        pageSize: self.pageSize,
                        page: self.pageNo
                    }
                    if (
                        self.filter.report_day[0] &&
                        self.filter.report_day[1]
                    ) {
                        para.report_day = JSON.stringify(self.filter.report_day)
                    }
                    let data = window.all.tool.rmEmpty(para)

                    let { url, method } = self.$api.third_game_report_list
                    self.$http({ method, url, data }).then(res => {
                        // console.log('列表👌👌👌👌: ', res)
                        if (res && res.code === '200'&&res.data) {
                            resolve(res.data.data)
                        }
                    })
                })
            }
            if (!self.total) return
            let totalPage = Math.ceil(self.total / 100)
            let list = []
            for (let i = 1; i <= totalPage; i++) {
                let currList = await getPageList(i) // 获取i页的内容
                list = list.concat(currList)
            }

            let chainName = window.all.tool.getChainName(self.$route.path)
            let file_name = `${chainName}`
            self.ExcelOut(list, file_name)
        },
        tofixedTwo(num) {
            if (!num) return 0
            return Number(num).toFixed(2)
        },
        updateNo() {
            this.getList()
        },
        updateSize() {
            this.pageNo = 1
            this.getList()
        },
        getList() {
            let para = {
                // report_day: this.filter.report_day,
                game_vendor_sign: this.filter.game_vendor_sign,
                pageSize: this.pageSize,
                page: this.pageNo
            }
            if (this.filter.report_day[0] && this.filter.report_day[1]) {
                para.report_day = JSON.stringify(this.filter.report_day)
            }
            let data = window.all.tool.rmEmpty(para)

            let { url, method } = this.$api.third_game_report_list
            this.$http({ method, url, data }).then(res => {
                // console.log('列表👌👌👌👌: ', res)
                if (res && res.code === '200') {
                    this.total = res.data.total
                    this.list = res.data.data
                }
            })
        }
    },
    mounted() {
        this.getList()
    }
}
</script>

<style scoped>
.mh5 {
    margin-left: 5px;
    margin-right: 5px;
}
.filter-row2 {
    padding-left: 10px;
    padding-bottom: 10px;
}
.mt20 {
    margin-top: 20px;
}
</style>