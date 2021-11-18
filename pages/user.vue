<template>
    <div>
        <Header />
        <div v-if="userInfo" class="userinfo">
            <p>昵称：{{ userInfo.username }}</p>
            <p>邮箱：{{ userInfo.email }}</p>

            <div v-if="Array.isArray(commentList) && commentList.length > 0" class="comment">
                <h2>评论列表：</h2>
                <ul class="comment-list">
                    <li v-for="item in commentList" :key="item.id" class="comment-item">
                        <p>文章：{{ item.article_info.title }}</p>
                        <p>评论内容：{{ item.content }}</p>
                        <p>评论时间：{{ item.created_at | formatTime }}</p>
                        <p>回复：
                            <div v-if="item.replys.length > 0" class="reply">
                                <div
                                    v-for="reply in item.replys"
                                    :key="reply.id"
                                    class="reply-item"
                                >
                                    <p>回复内容： {{reply.content}}</p>
                                    <p>回复时间： {{reply.created_at | formatTime }}</p>
                                </div>
                            </div>
                            <span v-else>无</span>
                        </p>
                    </li>
                    
                </ul>
                <div class="pagination">
                    <el-pagination
                        background
                        :current-page.sync="page"
                        layout="total, prev, pager, next"
                        :total="count"
                        @current-change="handleCurrentChange"
                    />
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import { mapState } from 'vuex'
import Header from '@/components/Header'
import { getCommentTarget } from '@/request/api/comment'
import moment from 'moment'


export default {
    name: "User",
    components: {
        Header
    },
    filters: {
        formatTime(value) {
            if (!value) return ''
            return moment(value).format("YYYY-MM-DD HH:mm")
        }
    },
    async asyncData(context) {

    },
    data() {
        return {
            page: 1,
            count: 0,
            commentList: []
        }
    },
    async fetch({ store }) {
        await store.dispatch('category/getCategoryData')
    },
    computed: {
        ...mapState({
            userInfo: state => state.user.userInfo
        })
    },
    head() {
        return {
            title: `  个人中心  - 7hds.com`,
            meta: [
                { name: 'keywords', content: 'hds,博客,何东树,blog,7hds,前端开发工程师,前端性能优化,JavaScript,css,html' },
                { name: 'description', content: 'hds博客 - 7hds.com，专注于前端开发技术，前端性能优化！' }
            ]
        }
    },


    mounted() {
        this.getComment()
        document.title = ` ${this.userInfo.username} - 个人中心  - 7hds.com`
    },
    methods: {
        async getComment() {
            const uid = this.userInfo && this.userInfo.id
            const [err, res] = await getCommentTarget({
                user_id: uid,
                is_replay: 1,
                is_article: 1,
                page: this.page
            })
            if (!err) {
                this.isLoad = true
                this.commentList = res.data.data.data
                this.count = res.data.data.meta.count
                console.log(this.commentList)
            }
        },
        // 点击数字
        async handleCurrentChange(page) {
            this.page = page
            await this.getComment()
            this.$scrollTo(0)
        },
    }
}
</script>

<style scoped lang="scss">
.userinfo {
    width: 1024px;
    margin: 32px auto;
    font-size: 14px;
}

.comment {
    margin-top: 70px;
    border-top: 1px solid ;
}
.comment-item {
    padding: 20px 0;
    border-bottom: 1px solid #f0f0f0;
}
.reply {
    padding-left: 20;
    .reply-item {
        padding:1px 2px;
        padding-left: 20px;
        background:#f1f1f1;
    }
}
</style>
