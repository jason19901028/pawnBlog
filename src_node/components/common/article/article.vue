<template>
	<div class="article" id="articleRippleWrap">
		<ul>
			<li class="item" v-for="(item, index) in currentArticleList" :key="index">
				<p class="mainTitle" @click="toArticlePage(item.ID)">
					<span>[{{item.original}}] </span>{{item.mainTitle}}</p>
				<ul class="tags">
					<li v-for="(tag, i) in item.tags" :key="i" @click="chooseTag(tag)">{{tag}}</li>
				</ul>
				<p class="intro" @click="toArticlePage(item.ID)" v-html="item.intro"></p>
				<div class="info">
					<div class="date">发表时间：{{item.date}}&nbsp;&nbsp;</div>
					<div class="column">|&nbsp;&nbsp;分类：
						<span>{{item.col}}</span>
					</div>
					<!-- 全部博客 -->
					<div class="more" @click="toEditDraft(item.ID, isDraft)" v-if="toEdit && isDraft">
						<span class="info">编辑</span>
					</div>
					<div class="delete" @click="toDeleteDraft(item.ID, index, isDraft)" v-if="toEdit && isDraft">
						<span class="info">删除</span>
					</div>
					<!-- 草稿箱 -->
					<div class="more" @click="toEditArticle(item.ID, isArticle)" v-if="toEdit && isArticle">
						<span class="info">编辑</span>
					</div>
					<div class="delete" @click="toDeleteArticle(item.ID, index, isArticle)" v-if="toEdit && isArticle">
						<span class="info">删除</span>
					</div>
				</div>
			</li>
		</ul>
		<div class="pagination">
			<button class="prev" @click="prevPage" v-show="allPageNum > 1">
				<i>&lt;</i>
				上一页
			</button>
			<button class="next" @click="nextPage" v-show="allPageNum > 1">
				下一页
				<i>&gt;</i>
			</button>
		</div>
	</div>
</template>

<script>
import { ripple } from '@/assets/script/common';

export default {
	data() {
		return {
			currentPage: 1,
			articleListCope: this.articleList
		}
	},
	props: {
		articleList: {
			type: Array,
			default() {
				return []
			}
		},
		toEdit: {
			type: Boolean,
			default: false
		},
		isDraft: {
			type: Boolean,
			default: false,
		},
		isArticle: {
			type: Boolean,
			default: false,
		}
	},
	computed: {
		//当前需要渲染文章列表
		currentArticleList() {
			return this.articleListCope.slice((this.currentPage - 1) * 6, this.currentPage * 6)
		},
		//总页面数
		allPageNum() {
			return Math.ceil(this.articleListCope.length / 6)
		}
	},
	watch: {
		articleList() {
			this.articleListCope = this.articleList
			//文章总列表有变化（切换了tab），重置curentPage
			this.currentPage = 1
		},
		currentArticleList() {
			//当前要渲染的列表有变化，重新实现ripple效果
			this.$nextTick(() => {
				ripple('articleRippleWrap');
			})
		}
	},
	methods: {
		nextPage() {
			if (this.currentPage >= this.allPageNum) {
				this.currentPage = this.allPageNum
				// 最后一页了，发射一个事件给父组件
				this.$emit('lastPage')
			} else {
				this.currentPage = ++this.currentPage
			}
		},
		prevPage() {
			if (this.currentPage <= 1) {
				this.currentPage = 1
				// 首页了，发射一个事件给父组件
				this.$emit('firstPage')
			} else {
				this.currentPage = --this.currentPage
			}
		},
		chooseTag(itemTag) {
			this.$emit('toTag', itemTag)
		},
		toArticlePage(id) {
			if (this.isDraft) {
				this.toEditDraft(id, true)
				return;
			}
			const userId = this.$route.query.userId || this.articleList[0].userId;
			if (userId === undefined) {
				this.$router.push({ path: '/article', query: { articleId: id } })
			} else {
				this.$router.push({ path: '/article', query: { userId: userId, articleId: id } })
			}
		},
		toEditDraft(id, isDraft) {
			if (!isDraft) {
				return;
			}
			this.$router.push({ name: 'BWriteBolg', params: { 'id': id, isArticle: false } })
		},
		toEditArticle(id, isArticle) {
			if (!isArticle) {
				return;
			}
			this.$router.push({ name: 'BWriteBolg', params: { 'id': id, isArticle: true } })
		},
		toDeleteArticle(id, index, isArticle) {
			if (!isArticle) {
				return
			}
			// 删除文章
			this.$http.post('/deleteArticle', { itemId: id }).then(function(res) {
				this.$emit('showMessage', res.body.ret_msg)
				this.articleListCope.splice(index, 1)
			}, function(res) {
				console.log(res);
			})
		},
		toDeleteDraft(id, index, isDraft) {
			if (!isDraft) {
				return
			}
			// 删除草稿
			this.$http.post('/deleteDraft', { itemId: id }).then(function(res) {
				this.$emit('showMessage', res.body.ret_msg)
				this.articleListCope.splice(index, 1)
			}, function(res) {
				console.log(res);
			})
		}
	},
}

</script>

<style lang="less">
@import '../../../assets/style/common.less';
@m26a69a : #26a69a;
@m009688 : #009688;
@mdedede : #dedede;
@m383838: #383838;
.article {
	width: 75%;
	float: left;
	position: relative;
	.item {
		padding: 34px 20px 20px;
		border-bottom: 1px solid @mdedede;
		transition: all .3s;
		&:hover {
			box-shadow: 0 5px 10px 2px rgba(0, 0, 0, .1);
		}
	}
	.mainTitle {
		font-size: 20px;
		line-height: 34px;
		color: #666;
		cursor: pointer;
		&:hover {
			text-decoration: underline;
			color: @m26a69a;
		}
		span {
			color: @m26a69a;
		}
	}
	.subTitle {
		font-size: 16px;
		line-height: 30px;
		color: @m383838;
	}
	ul.tags {
		.clearfixMixin();
		margin-top: 10px;
		li {
			float: left;
			font-size: 12px;
			padding: 5px 10px;
			margin-right: 10px;
			color: #fff;
			background-color: #00b5a3;
			border-radius: 6px;
			cursor: pointer;
			transition: all .5s;
			&:hover {
				background-color: #f76689;
			}
		}
	}
	.intro {
		font-family: 'sourcesanspro', 'Helvetica Neue', Arial, sans-serif;
		margin-top: 20px;
		font-size: 14px;
		color: #828282;
		line-height: 24px;
		height: 72px;
		overflow: hidden; // .txt-cut(3);
		cursor: pointer;
		&:hover {
			text-decoration: underline;
		}
	}
	.info {
		font-family: 'sourcesanspro', 'Helvetica Neue', Arial, sans-serif;
		margin-top: 30px;
		font-size: 14px;
		line-height: 20px;
		.clearfixMixin();
		.date,
		.column {
			float: left;
			font-size: 12px;
			color: #828282;
		}
		.more,
		.delete {
			float: right;
			border-radius: 2px;
			color: @m26a69a;
			cursor: pointer;
			margin-left: 8px;
			&:hover {
				color: #0d7b71;
			}
		}
	}
	.pagination {
		margin: 50px 0 80px;
		padding: 0 20px;
		.clearfixMixin();
		button {
			background-color: transparent;
			color: #2e9c92;
			font-size: 14px;
			line-height: 30px;
			padding: 0 10px;
			border-radius: 2px;
			cursor: pointer;
			transition: all .3s;
		}
		.prev:hover,
		.next:hover {
			color: #fff;
			background-color: #2e9c92;
		}
		.prev {
			float: left;
		}
		.next {
			float: right
		}
		i {
			font-family: sans-serif;
			font-style: normal
		}
	}
}
</style>
