<template>
	<div class="updatetips" ref="updatetips">
		<transition name="fade" @after-leave="hideUpdate" @before-enter="showUpdate">
			<div class="update_mask" v-show="showInfo"></div>
		</transition>
		<span class="updatetime">{{updateInfo.updatetime}}</span>
		<transition name="scale">
			<div class="updatetips_info" v-show="showInfo">
				<div class="img_bg">
					<!-- <img src="http://oiq8j9er1.bkt.clouddn.com/update_bg.jpg" alt="未曾遗忘的青春"> -->
					<h3 class="title">更新提示</h3>
					<span class="update_version">v{{updateInfo.version}}</span>
				</div>
				<div class="content" v-html="updateInfo.disc">
				</div>
				<div class="update_select">
					<input type="button" @click="unUpdate" class="giveup_update" value="暂不体验">
					<input type="button" @click="update" class="go_update" value="体验新版">
				</div>
			</div>
		</transition>
	</div>
</template>

<script>
  import {Cookie} from '@/common/js/global.js'
  import fecth from './../../../utils/fecth.js'
  export default {
  	data () {
  		return {
  			showInfo: false,
  			updateInfo: {}
  		}
  	},
  	methods: {
  		unUpdate () {
  			this.showInfo = false
  			// 添加cookie方便以后过期再弹窗
  			Cookie.setCookie('update', this.updateInfo.version, 60 * 60 * 2 * 1000)
  		},

  		// 更新网页  包括默认配置  以及 清除缓存刷新页面
  		update () {
  			this.showInfo = false
  			window.localStorage.setItem('web_version', this.updateInfo.version)
  			Cookie.setCookie('update', this.updateInfo.version, 60 * 60 * 2 * 1000)
  			location.href = location.href + (location.href.indexOf('?') > -1 ? '&' : '?') + 'v=' + this.updateInfo.version
  		},

  		showUpdate () {
  			this.$refs.updatetips.style.display = 'block'
  		},

  		hideUpdate () {
  			this.$refs.updatetips.style.display = 'none'
  		},

  		isNeedUpdate () {
  			const _this = this
  			const url = 'http://www.daiwei.org/vue/server/home.php?inAjax=1&do=isNeedUpdate'
			fecth.get(url).then((res) => {
				_this.updateInfo = res.data[0]
				let versionStorage = window.localStorage.getItem('web_version')
				// 本地无存储号  必须弹窗提示更新
				// 本地有存储的版本号  和 线上的不一样  弹窗
				// 如果设置2小时之后再弹窗  则超出之后再弹窗更新
				// 如果在2小时内又更新了一个版本   则还是需要弹窗更新
				if (versionStorage !== null) {
					console.log('有versionStorage')
					if (Cookie.getCookie('update') !== null) {
						console.log('有cookie')
						if (Cookie.getCookie('update') !== _this.updateInfo.version) {
							console.log('cookie版本不同 --- 弹窗')
							_this.showInfo = true
						} else {
							console.log('cookie版本相同 --- 不弹窗')
							_this.showInfo = false
						}
					} else {
						if (versionStorage !== _this.updateInfo.version) {
							console.log('没有cookie时候  versionStorage版本不同 --- 弹窗')
							_this.showInfo = true
						} else {
							console.log('versionStorage版本相同')
							_this.showInfo = false
						}
					}
				} else {
					console.log('没有versionStorage')
					window.localStorage.setItem('web_version', this.updateInfo.version)
					Cookie.setCookie('update', this.updateInfo.version, 60 * 60 * 2 * 1000)
				}
			}, (err) => {
				alert(err)
			})
  		}
  	},
  	mounted () {
  		this.isNeedUpdate()
  	}
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
	$update_primary_color = rgba(0,0,0,0.5)
	$bginfo = rgba(0,0,0,0.5)
	$text_title = rgba(255,255,255,0.6)
	.updatetips
		position:fixed
		top:0
		left:0
		bottom:0
		right:0
		z-index:11
		font-weight:500
		display:none
		.updatetime
			position:absolute
			bottom:30px
			right:30px
			color:$text_title
			font-size:12px
		.update_mask
			position:absolute
			top:0
			left:0
			bottom:0
			right:0
			background:rgba(12,14,27,0.98)
			&.fade-enter-to,&.fade-leave-to
				transition:all 0.5s
			&.fade-enter,&.fade-leave-to
				opacity:0
		.updatetips_info
			position:absolute
			top:46%
			left:50%
			bottom:0
			right:0
			// background:$update_primary_color
			width:100%
			max-width:414px
			padding: 0 15px
			height:300px
			transform:translate(-50%,-50%) scale(1)
			border-radius: 4px
			overflow:hidden
			display:flex
			flex-direction:column
			justify-content:center
			&.scale-enter-to,&.scale-leave-to
				transition:all 0.3s
			&.scale-enter
				transform:translate(-50%, -30%)
				opacity:0
			&.scale-leave-to
				opacity:0
			.img_bg
				width:100%
				height:auto
				min-height:70px
				position:relative
				border-radius: 4px 4px 0 0
				.title
					color:rgba(255,255,255,0.6)
					padding: 0 15px
					text-align:center 
					margin:0
				img
					width:100%
					height:auto
					border-radius: 4px
				.update_version
					position:absolute
					right:30px
					top:40px
					color:$text_title
					font-size:14px
					font-weight:600
			.content
				width:100%
				height:120px
				padding: 0 35px
				line-height:1.7
				font-size: 14px
				overflow:hidden
				color:$text_title
				font-weight:400
				box-sizing: border-box;
			.update_select
				width:250px
				height:36px
				font-size:0
				position:absolute
				left:50%
				transform:translate3d(-50%,0,0)
				bottom:0
				border-radius:0 0 4px 4px
				// border-top:1px solid $text_title 
				background:transparent
				input
					width:50%
					height:100%
					font-size:14px
					height:36px
					line-height:36px
					border:0 none
					outline:0 none
					// background:$update_primary_color
					cursor:pointer
					-webkit-appearance:none
					outline:none
					background:transparent
					&.giveup_update
						border-radius:0 0 0 4px
						color: $text_title
						// border-right:1px solid $text_title  
					&.go_update
						border-radius:0 0 4px 0
						color: $text_title 
						// background:$update_primary_color
					&:hover{
						color: rgba(255,255,255,0.9)
					}
</style>