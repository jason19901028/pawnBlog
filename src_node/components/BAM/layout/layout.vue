<template>
	<div class="BAMCom">
		<v-BAMSide :userInfo="userInfo"></v-BAMSide>
		<router-view @showMessage="showMessage" :draftId="draftId" :articleId="articleId"></router-view>
		<v-Message :messageShow="messageShow" :sendMessage="sendMessage"></v-Message>
	</div>
</template>

<script>
import BAMSide from '@/components/BAM/side/side';
import Message from '@/components/common/Message/Message';
export default {
	data() {
		return {
			userInfo: {
				avatar: 'http://www.fepawn.com/upload/avatar/default.gif'
			},
			messageShow: false,
			sendMessage: '',
			draftId: 0,
			articleId: 0,
		}
	},
	mounted() {
		this._initUserInfo()
	},
	watch: {
		$route(to, from) {
			if (to.path.indexOf('/BAM/BWriteBolg') > -1) {
				if (to.params.isArticle === true) {
					this.articleId = to.params.id
					this.draftId = 0
				} else if (to.params.isArticle === false) {
					this.draftId = to.params.id
					this.articleId = 0
				} else {
					this.articleId = 0
					this.draftId = 0
				}
			}
		}
	},
	methods: {
		showMessage(msg, timeout) {
			var timeout = timeout || 1500
			if (!this.messageShow) {
				const _this = this
				this.messageShow = true;
				this.sendMessage = msg
				setTimeout(function() {
					_this.messageShow = false;
				}, timeout)
			}
		},
		_initUserInfo() {
			this.$http.get('/initUserInfo').then(function(res) {
				if (res.body.ret_code === "000") {
					this.userInfo = res.body.data
				} else if (res.body.ret_code = "002") {
					//没有session，未登录（未按步骤操作）
					this.$router.push({ path: '/login' })
				} else {
					this.showMessage('系统错误，请稍后再试')
				}
			}, function(err) {
				console.log(err);
			})
		}
	},
	components: {
		'v-BAMSide': BAMSide,
		'v-Message': Message
	}
}
</script>

<style scoped>
.BAMCom {
	position: absolute;
	top: 50px;
	left: 0;
	right: 0;
	bottom: 0;
	background-color: #fff;
}
</style>



