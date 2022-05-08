<template>
	<view class="content">
		<!-- 输入区域 -->
		<view class="home-input">
			<view class="hi-input">
				<textarea 
					name="fantan" 
					cols="30" 
					rows="10" 
					:maxlength="5000" 
					placeholder="请将要处理的信息粘贴于此"
					v-model="fantanText"
				></textarea>
			</view>
			<view class="hi-btn">
				<button type="primary" @click="startCompare">比对</button>
			</view>
		</view>
		<!-- 结果区域 -->
		<view class="result-area">
			<uni-card title="基础卡片">
				<template v-slot:title>
					<view class="card-title">
						<view class="title-left">
							<text>结果：</text>
						</view>
						<view class="title-btn" @click="gotoAddPage">
							<uni-icons type="plusempty" size="26"></uni-icons>
						</view>
					</view>
				</template>
				<scroll-view scroll-y="true"  v-if="compareResult.length > 0">
					<view class="result-item" v-for="(item, index) in compareResult" :key="index">
						<text class="res-index">{{index + 1}}.</text>
						<view class="res-name">{{item.name || ''}}</view>
						<view class="res-info" :style="{color: computeColor(item.status)}">{{item.info || ''}}</view>
					</view>
				</scroll-view>
				<view class="no-result" v-else>
					暂无信息，请点击“＋”增加群组成员
				</view>
			</uni-card>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				fantanText: '',
				members: [],
				compareResult: [],
			}
		},
		onShow() {
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
					console.log('members, ', this.members)
					// 如果从另一个页面返回时有接龙信息，则立即进行比较
					if (!this.noInfoHandler()) {
						this.compare()
					}
				},
				fail: (reason) => {
					this.members = []
				}
			})
		},
		watch: {
			// 每当输入接龙信息变为空，就要将状态设置为 －1
			fantanText(newVal) {
				if (!newVal) {
					this.noInfoHandler()
				}
			}
		},
		methods: {
			gotoAddPage() {
				uni.navigateTo({
					url: '/pages/add/add',
					animationType:"slide-in-right"
				})
			},
			// 没有接龙信息时将状态设置为 －1，并返回是否有信息
			noInfoHandler() {
				const fantanText = this.fantanText?.trim() || ''
				if (!fantanText) {
					this.compareResult = this.members.map(mem => {
						return {
							status: -1,
							name: (mem.nicknames && mem.nicknames[0]) || '无名氏',
							info: '暂无信息'
						}
					})
					return true
				} else {
					return false
				}
			},
			// 点击比对
			startCompare() {
				if (this.noInfoHandler()) {
					uni.showToast({
						icon: 'none',
						title: '请填写信息哦！'
					})
					return
				} 
				this.compare()
			},
			// 比对
			compare() {
				if (this.noInfoHandler()) {
					return
				} 
				console.log(this.fantanText)
				const splitInfo = this.fantanText.split('\n').filter(fragment => /^\d.*/.test(fragment))
				console.log(splitInfo)
				const fantanInfoObjectList = splitInfo.map(str => {
					const nameReg = /(?<=^\d+\.\s).*(?=\s\S*$)/
					const infoReg = /\S*$/
					const userNameMatch = str.match(nameReg)
					const userName = userNameMatch ? userNameMatch[0] : ''
					const infoMatch = str.match(infoReg)
					const info = infoMatch ? infoMatch[0] : ''
					return {
						name: userName,
						info: info,
					}
				})
				console.log(fantanInfoObjectList)
				const result = this.members.map(mem => {
					const nickNames = mem.nicknames || []
					let info = '尚未完成'
					let hadComplete = nickNames.some(nick => {
						return fantanInfoObjectList.some(obj => {
							if (obj.name === nick) {
								info = obj.info
								return true
							} else {
								return false
							}
						})
					})
					return {
						status: hadComplete ? 1 : 0,
						name: nickNames[0] || '无名氏',
						info: info
					}
				})
				console.log(result)
				let hadCompleteList = result?.filter(item => item.status === 1) || []
				let hadNotCompleteList = result?.filter(item => item.status === 0) || []
				this.compareResult = [...hadNotCompleteList, ...hadCompleteList]
			},
			computeColor(status) {
				if (status === 1) {
					return 'green'
				} else if (status === 0) {
					return 'red'
				} else if (status === -1) {
					return '#999999'
				}
			}
		}
	}
</script>

<style lang="scss">
	$border-b: 1px solid #EBEEF5;
	.content {
		display: flex;
		flex-direction: column;
		height: 100%;
		.home-input {
			.hi-input {
				border: 1rpx solid $uni-border-color;
				textarea {
					width: 100%;
					height: 270rpx;
					padding: 5rpx;
					box-sizing: border-box !important;
				}
			}
			.hi-btn {
				padding: $vertical-padding 0;
			}
		}
		.result-area {
			flex: 1;
			min-height: 0;
			::v-deep .uni-card {
				margin: 0 !important;
				height: 100%;
				box-shadow: none !important;
				padding: 0 !important;
				.uni-card__content {
					width: 100%;
					height: calc(100% - 86rpx);
					padding: 15rpx 0 !important;
					scroll-view {
						height: 100%;
						.result-item {
							display: flex;
							height: 60px;
							border-bottom: $border-b;
							align-items: center;
							padding: 5rpx 0;
							.res-index {
								width: 40px;
								padding-left: 10rpx;
							}
							.res-name {
								flex: 1;
								padding: 0 10rpx;
							}
							.res-info {
								flex: 1;
								padding: 0 10rpx;
							}
						}
					}
					.no-result {
						display: flex;
						justify-content: center;
						align-items: center;
						height: 100%;
					}
				}
			}
			.card-title {
				height: 86rpx;
				display: flex;
				justify-content: space-between;
				align-items: center;
				border-bottom: $border-b;
				padding: 0 10rpx;
				.title-left {
					padding: 0 5rpx;
					font-size: 18px;
				}
				.card-btn {
					
				}
			}
		}
		.goto-btn {
			padding: $vertical-padding 0;
		}
	}
</style>
