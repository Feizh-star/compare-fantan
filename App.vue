<script>
	import pageAnimation from './components/page-animation'
	export default {
		mixins: [pageAnimation],
		onLaunch: function() {
			console.log('App Launch')
		},
		onShow: function() {
			console.log('App Show')
		},
		onHide: function() {
			console.log('App Hide')
		}
	}
</script>

<style lang="scss">
	/*每个页面公共css */
	* {
		box-sizing: border-box;
	}
	uni-page-body {
		width: 100%;
		height: 100%;
		padding: $page-padding;
	}
</style>
