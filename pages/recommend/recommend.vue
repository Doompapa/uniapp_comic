<template>
	<view class="wrap">
		<view class="status_bar">
			<!-- 这里是状态栏 -->
		</view>
		<view class="search_bar" @click="gotosearch">
			<u-search placeholder="搜索作品名" :clearabled="true" :show-action="false" bg-color="#ffffff" disabled="false"></u-search>
		</view>
		<u-waterfall :flowList="booklist">
			<template v-slot:left="{leftList}">
				<view class="carton_box" v-for="(item, index) in leftList" :key="index" @click="gotocomicdetails(item.id,2)">
					<!-- 警告：微信小程序不支持嵌入lazyload组件，请自行如下使用image标签 -->
					<!-- #ifndef MP-WEIXIN -->
					<u-lazy-load threshold="-450" border-radius="10" :image="item.cover_url" :index="index"></u-lazy-load>
					<!-- #endif -->
					<!-- #ifdef MP-WEIXIN -->
					<view class="img_wrap">
						<image class="carton_image" :src="item.cover_url" mode="widthFix" lazy-load=true></image>
					</view>
					<!-- #endif -->
					<view class="book_title">
						{{item.book_name}}
					</view>
				</view>
			</template>
			<template v-slot:right="{rightList}">
				<view class="carton_box" v-for="(item, index) in rightList" :key="index" @click="gotocomicdetails(item.id,2)">
					<!-- #ifndef MP-WEIXIN -->
					<u-lazy-load threshold="-450" border-radius="10" :image="item.cover_url" :index="index"></u-lazy-load>
					<!-- #endif -->
					<!-- #ifdef MP-WEIXIN -->
					<view class="img_wrap">
						<image class="carton_image" :src="item.cover_url" mode="widthFix" lazy-load=true></image>
					</view>
					<!-- #endif -->
					<view class="book_title">
						{{item.book_name}}
					</view>
				</view>
			</template>
		</u-waterfall>
		<u-loadmore bg-color="rgb(255, 255, 255)" :status="loadStatus" @loadmore="getbooklist"></u-loadmore>
	</view>
</template>

<script>
	import md5 from '@/md5.js'
	export default {
		data() {
			return {
				loadStatus: 'loadmore',
				booklist: [],
				startlem: 0,
				pageSize: 6,
				userInfo:{}
			}
		},
		onLoad() {
			this.getbooklist();
			this.userInfo = this.$store.state;
		},
		onReachBottom() {
			this.loadStatus = 'loading';
			// 模拟数据加载
			setTimeout(() => {
				this.getbooklist();
				this.loadStatus = 'loadmore';
			}, 1000)
		},
		methods: {
			getapi() {
				var timestamp = Date.parse(new Date());
				var api_key = this.api_key;
				return [timestamp, md5(api_key + timestamp)];
				//返回api_key+时间戳md5加密
			},
			async getbooklist() {
				var key = await this.getapi();
				uni.request({
					url: this.url_config+'app/tags/getBookList',
					data: {
						time: key[0],
						token: key[1],
						startItem: this.startlem,
						pageSize: this.pageSize
					},
					success: (res) => {
						console.log(res.data)
						this.booklist = this.booklist.concat(res.data.books);
						this.startlem = this.startlem + this.pageSize;
					}
				});
			},
			gotocomicdetails(id, uid) {
				uni.navigateTo({
					url: '../comicdetails/comicdetails?id=' + id + '&uid=' + uid
				})
			},
			gotosearch() {
				uni.navigateTo({
					url: '../search/search'
				})
			}
		}
	}
</script>

<style lang="scss" scoped>
	.wrap {
		background-color: #f7f9fa;
	}
	.status_bar {
		height: var(--status-bar-height);
		width: 100%;
	}
	
	.search_bar {
		width: 100%;
		margin-top: 4rpx;
	}
	.carton_box {
		border-radius: 5rpx;
		margin: 12rpx;
		background-color: #FFFFFF;
		padding: 8rpx;
	}

	.carton_image {
		width: 100%;
		border-radius: 5rpx;
	}

	.book_title {
		font-size: 30rpx;
		margin-top: 5rpx;
		color: $u-main-color;
	}
</style>
