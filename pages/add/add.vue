<template>
	<view class="page-add">
		<view class="add-main">
			<scroll-view 
				scroll-y="true"
			>
				<uni-card :title="item.nicknames[0]" v-for="(item) in members" :key="item.id">
					<template v-slot:title>
						<view class="card-title">
							<view class="title-left">
								<text>{{item.nicknames[0] || '新成员'}}</text>
							</view>
							<view class="title-btn">
								<button type="warn" size="mini" @click="deleteMember(item.id)">删除</button>
							</view>
						</view>
					</template>
					<view class="card-main">
						<view class="input-row" v-for="(nick, index) in item.nicknames" :key="index">
							<label class="row-label">昵称{{ index + 1 }}：</label>
							<input class="row-input" v-model="item.nicknames[index]" type="text" placeholder="请输入昵称">
							<view class="row-btns">
								<uni-icons 
									type="minus-filled" 
									size="26" 
									style="color: #e64340;" 
									@click="deleteNick(item.nicknames, index)"
								></uni-icons>
								<uni-icons 
									v-if="index === item.nicknames.length - 1" 
									type="plus-filled" 
									size="26" style="color: #41b783;" 
									@click="addNick(item.nicknames)"
								></uni-icons>
								<uni-icons 
									v-else 
									type="plus-filled" 
									size="26" 
									style="opacity: 0;"
								></uni-icons>
							</view>
						</view>
					</view>
				</uni-card>
			</scroll-view>
		</view>
		<view class="add-btn">
			<button type="primary" @click="addMember">添加一位成员</button>
			<view style="height: 5px;"></view>
			<button type="primary" class="green-btn" @click.stop="saveInfo">保存成员信息</button>
		</view>
	</view>
</template>

<script>
	const info = {
		list: [
			{
				id: '123435640',
				nicknames: ['张三', '美梦']
			},
			{
				id: '123435641',
				nicknames: ['李四', '好吃']
			}
		]
	}
	export default {
		data() {
			return {
				members: [],
			};
		},
		onShow(params) {
			uni.getStorage({
				key: 'user_info',
				success: (res) => {
					const userInfo = res.data
					this.members = userInfo.list
					this.members.forEach(mem => {
						if (!mem.nicknames) {
							mem.nicknames = []
						}
						if (mem.nicknames.length === 0) {
							mem.nicknames.push('')
						}
					})
				},
				fail: (reason) => {
					this.members = []
				}
			})
			this.isLastClick = true
		},
		onBackPress() {
			console.log(this.isLastClick)
			if (!this.isLastClick) {
				this.saveInfo()
			}
		},
		watch: {
			members: {
				deep: true,
				handler(newVal) {
					this.isLastClick = false
				}
			}
		},
		methods: {
			addMember() {
				this.members.push({
					id: new Date().getTime(),
					nicknames: ['']
				})
			},
			deleteMember(id) {
				const memberIndex = this.members.findIndex(mem => mem.id === id)
				this.members.splice(memberIndex, 1)
			},
			addNick(nickList) {
				nickList.push('')
			},
			deleteNick(nickList, index) {
				nickList.splice(index, 1)
			},
			saveInfo() {
				let members = JSON.parse(JSON.stringify(this.members))
				members = members.filter(mem => {
					if (!mem.nicknames) return false
					mem.nicknames = mem.nicknames.filter(name => name.length > 0)
					return mem.nicknames.length > 0
				})
				const userInfo = {
					list: members
				}
				console.log('userInfo', userInfo)
				uni.setStorage({
					key: 'user_info',
					data: userInfo,
					success: (res) => {
						console.log('save success', res)
						uni.showToast({
							mask: true,
							icon: 'success',
							title: '保存成功！'
						})
						this.isLastClick = true
					},
					fail: (error) => {
						console.log('save error', error)
						uni.showToast({
							mask: true,
							icon: 'error',
							title: '保存失败！'
						})
					}
				})
			}
		},
		clearIsLastClickStatus() {
			this.isLastClick = false
		}
	}
</script>

<style lang="scss">
	.page-add {
		display: flex;
		flex-direction: column;
		height: 100%;
		.add-main {
			flex: 1;
			min-height: 0;
			margin-left: - $page-padding;
			margin-right: - $page-padding;
			scroll-view {
				height: 100%;
			}
			::v-deep .uni-card {
				margin: 10rpx !important;
				box-shadow: none !important;
				padding: 0 !important;
				.uni-card__content {
					width: 100%;
					height: calc(100% - 86rpx);
				}
			}
			.card-title {
				height: 86rpx;
				display: flex;
				justify-content: space-between;
				align-items: center;
				border-bottom: 1rpx solid #EBEEF5;
				padding: 0 10rpx;
				.title-left {
					padding: 0 5rpx;
					font-size: 18px;
					flex: 1;
					min-width: 0;
					text {
						padding-right: 20rpx;
						display: block;
						width: 100%;
						min-width: 0;
						overflow: hidden;
						white-space: nowrap;
						text-overflow: ellipsis;
					}
				}
				.title-btn {
					width: fit-content;
					height: 100%;
					display: inline-flex;
					align-items: center;
				}
			}
			.card-main {
				.input-row {
					// border-bottom: 1px solid #EBEEF5;
					display: flex;
					height: 38px;
					align-items: center;
					// <label class="row-label" for="input1">昵称1：</label>
					// <input class="row-input" type="text">
					// <view class="row-btns">
					.row-label {
						width: 60px;
					}
					.row-input {
						flex: 1;
						border: 1px solid $uni-border-color;
						height: 32px;
						padding-left: 5px;
						padding-right: 5px;
					}
					.row-btns {
						width: 60px;
						padding: 5px;
						display: inline-flex;
						justify-content: space-around;
						align-items: center;
					}
				}
			}
		}
		.add-btn {
			padding: $vertical-padding 0;
			padding-bottom: 0;
			.green-btn {
				background-color: #41b783;
				&.button-hover {
					color: rgba(255, 255, 255, .6);
					background-color: #57b792;
				}
			}
		}
	}
</style>
