<template>
    <div class="container">
        <!-------------------- 分类管理 ------------------------->
        <div class="filter p10">
            <ul class="left">
                <li>
                    <span>分类名称</span>
                    <!-- <Input class="w100" v-model="filter.name" /> -->
                    <Select v-model="filter.name" input :options="sort_opt"></Select>
                </li>
                <li>
                    <span>厂商状态</span>
                    <Select v-model="filter.status" :options="status_opt"></Select>
                </li>
                <li>
                    <button class="btn-blue" @click="getList">查询</button>
                </li>
            </ul>
        </div>
        <div class="table mt10">
            <Table :headers="headers" :column="list">
                <template v-slot:item="{row,idx}">
                    <td>{{(pageNo-1)*pageSize+idx+1}}</td>
                    <td>{{row.name}}</td>
                    <td>{{row.last_editor?row.last_editor.name:'--'}}</td>
                    <td>{{row.updated_at}}</td>
                    <!-- <td
                        :class="[row.status===1?'green':'red']"
                    >{{row.status===1?'开启':row.status===0?'关闭':'---?'}}</td>-->
                    <td>
                        <Switchbox v-model="row.status" @update="statusSwitch(row)" />
                    </td>
                    <!-- <td>
                        <button
                            :class="[row.status?'btns-red':'btns-green']"
                            @click="statusSwitch(row)"
                        >{{row.status===1?'禁用':'启用'}}</button>
                    </td>-->
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
        <Modal
            :show.sync="mod_show"
            :title="mod_title"
            :content="mod_cont"
            @cancel="mod_show=false"
            @confirm="modConf"
        ></Modal>
    </div>
</template> <script>
export default {
    name: 'CashFlowSort',
    data() {
        return {
            filter: {
                name: '',
                status: ''
            },
            sort_opt: [],
            status_opt: [
                { label: '全部', value: '' },
                { label: '开启', value: '1' },
                { label: '关闭', value: '0' }
            ],
            headers: [
                '编号',
                '分类名称',
                '最后更新人',
                '最后更新时间',
                '状态'
                // '操作'
            ],
            list: [],
            status_obj: {
                '1': '禁用',
                '0': '启用'
            },
            total: 0,
            pageNo: 1,
            pageSize: 25,

            mod_show: false,
            mod_title: '',
            mod_cont: '',
            curr_row: {}
        }
    },
    methods: {
        getJSONOpt() {
            window.all.tool.getJsonOpt('system_finance_type').then(res => {
                if (res && Array.isArray(res)) {
                    let arr = res.map(item=>{
                        return {
                            label: item.name,
                            value: item.name
                        }
                    })
                    this.sort_opt = [{label:'全部',value:''}].concat(arr)
                }
            })
        },
        statusSwitch(row) {
            this.curr_row = row
            // this.mod_show = true
            // if (row.status === 1) {
            //     this.mod_title = '禁用'
            //     this.mod_cont = '是否确认禁用该分类!'
            // } else {
            //     this.mod_title = '启用'
            //     this.mod_cont = '是否确认启用该分类!'
            // }
            this.modConf()
        },
        modConf() {
            let data = {
                status: this.curr_row.status ? 1 : 0,
                id: this.curr_row.id
            }

            let { url, method } = this.$api.finance_sort_set
            this.$http({ method, url, data }).then(res => {
                if (res && res.code === '200') {
                    this.$toast.success(res.message)
                    // this.mod_show = false
                }
                this.getList()
            })
        },
        // getList: window.all.tool.debounce(function() {
        // console.log('执行')
        getList() {
            let self = this
            let para = {
                name: this.filter.name,
                status: this.filter.status,
                pageSize: this.pageSize,
                page: this.pageNo
            }
            let params = window.all.tool.rmEmpty(para)
            let { url, method } = this.$api.finance_sort_list
            this.$http({
                method: method,
                url: url,
                params: params
            }).then(res => {
                if (res && res.code === '200') {
                    self.total = res.data.total
                    this.list = res.data.data
                }
            })
        },
        updateNo() {
            this.getList()
        },
        updateSize() {
            this.pageNo = 1
            this.getList()
        }
    },
    mounted() {
        this.getJSONOpt()
        this.getList()
    }
}
</script>

<style scoped>
</style>