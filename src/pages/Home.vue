<template>
	<div id="home">
		<template>
			<tipsDialog :msg="msgTips" ref="dialog"></tipsDialog>
			<div class="disconnect" v-show="isconnected">
				<i class="icon"></i>
				<span>交易、行情未连接，网络恢复时会自动连接！</span>
			</div>
			<topbar title="行情" :colorName="bg"></topbar>
			<button class="help" @tap="toHelp"></button>
			<!-- <button id="refresh" :class="{dynamic: isdynamic == true, static: isdynamic == false}" @tap="refresh"></button> -->
			<i class="icon_connected" v-show="iconIsconnected"></i>
			<div class="page_cont">
				<div id="selectbar">
					<ul>
						<li @tap="selectClass" class="current">国际期货</li>
						<!--<li @tap="selectClass">亚洲指数</li>-->
					</ul>
				</div>
				<div id="datalist">
					<ul class="head">
						<li>
							<ol>
								<li>合约名称</li>
								<li>成交量</li>
								<li>最新价</li>
								<li @tap="switchEvent">{{title}}<i class="icon_sj"></i></li>
							</ol>
						</li>
					</ul>
					<ul class="cont">
						<template v-for="(v,i) in Parameters">
							<li class="list cl" @tap='toDetail'>
								<ol>
									<li>
										<h5>{{v.CommodityName}}</h5>
										<h5>{{v.CommodityNo}}{{v.MainContract}}</h5>
									</li>
									<li>{{v.LastQuotation.TotalVolume}}</li>
									<li :class="{green: v.LastQuotation.ChangeRate < 0, red: v.LastQuotation.ChangeRate > 0}">{{v.LastQuotation.LastPrice | fixNum2(v.DotSize)}}</li>
									<li :class="{green: v.LastQuotation.ChangeRate < 0, red: v.LastQuotation.ChangeRate > 0}" v-show="isswitch">{{v.LastQuotation.ChangeRate | fixNum}}%</li>
									<li :class="{green: v.LastQuotation.ChangeRate < 0, red: v.LastQuotation.ChangeRate > 0}" v-show="!isswitch">{{v.LastQuotation.ChangeValue | fixNum2(v.DotSize)}}</li>
								</ol>
							</li>
						</template>
						<p class="tips">没有更多的合约可加载啦~</p>
						<div class="empty"></div>
					</ul>
					
				</div>

			</div>
		</template>
		<guide v-if='guideshow'></guide>
		<novice v-if="helpshow"></novice>
	</div>

</template>

<script>
	import tipsDialog from '../components/tipsDialog.vue'
	import topbar from '../components/Topbar.vue'
	import guide from './Guide.vue'
	import novice from './noviceQuote.vue'
	import VueNativeSock from 'vue-native-websocket'
	import { mapMutations,mapActions } from 'vuex'
	import pro from '../assets/common.js'
	export default {
		name: 'home',
		data() {
			return {
				time: 3,
				msg: '',
				title: '涨跌幅',
				isdynamic: true,
				isswitch: true,
				colors: '',
				isconnected: false,
				iconIsconnected: false,
				arr: ''
			}
		},
		filters:{
			fixNum:function(num){
				return num.toFixed(2);
			},
			fixNum2:function(num,dotsize){
				dotsize=dotsize;
				return num.toFixed(dotsize);
			}
		},
		components: {topbar, guide, tipsDialog, novice},
		computed: {
			msgTips: function(){
				return this.msg;
			},
			bg: function(){
				if(this.colors) return this.colors;
			},
			Parameters(){
				return this.$store.state.market.Parameters;
			},
			quoteIndex(){
				return this.$store.state.market.quoteIndex;
			},
			quoteColor(){
				return this.$store.state.market.quoteColor;
			},
			guideshow(){
				return this.$store.state.isshow.guideshow;
			},
			helpshow(){
//				if(this.$store.state.isshow.helpshow == true){
//					$("body").css({'overflow': 'hidden'});
//				}
				return this.$store.state.isshow.helpshow;
			},
			quoteConnectedMsg(){
				return this.$store.state.market.quoteConnectedMsg;
			},
			isBack: function(){
				return this.$route.query.isBack;
			},
			PATH: function(){
				return this.$store.getters.PATH;
			},
			tradeLoginSuccessMsg(){
				return this.$store.state.market.tradeLoginSuccessMsg;
			}
		},
		watch: {
			quoteIndex: function(n, o){
				if(this.quoteColor == 'red'){
					$("#datalist>.cont>li").eq(n).addClass("bgred");
					setTimeout(function(){
						$("#datalist>.cont>li").eq(n).removeClass("bgred");
					}, 500);
				}else{
					$("#datalist>.cont>li").eq(n).addClass("bggreen");
					setTimeout(function(){
						$("#datalist>.cont>li").eq(n).removeClass("bggreen");
					}, 500);
				}
			},
			quoteConnectedMsg: function(n, o){
				if(n && this.guideshow == false){
					this.$refs.dialog.isShow = true;
					this.msg = n.slice(0,-1);
					setTimeout(function(){
						this.isdynamic = false;
					}.bind(this),1000);
				}
			},
			guideshow: function(n, o){
				if(n == false){
					this.$refs.dialog.isShow = true;
					this.msg = '行情服务器连接成功';
				}
			},
			isBack: function(n, o){
				if(n == 1){
					window.location.reload();
				}
			},
			$route:function(){
				if(this.$route.name == 'home'){
					this.$store.state.menuType = 2;
				}
			}
		},
		methods: {
			...mapActions([
				'initQuoteClient'
			]),
			refresh: function(e) {
				if(pro.netIsconnected()){
					window.location.reload();
				}else{
					this.$refs.dialog.isShow = true;
					this.msg = '网络未连接，行情、交易不能刷新';
				}
			},
			toHelp: function(){
				this.$router.push({path: '/help'});
			},
			selectClass: function(e) {
				$(e.target).addClass('current').siblings('li').removeClass('current');
			},
			switchEvent: function(e){
				if($(e.currentTarget).text() == '涨跌幅'){
					this.title = '涨跌额';
					this.isswitch = false;
				}else{
					this.title = '涨跌幅';
					this.isswitch = true;
				}
			},
			toDetail: function(a) {
				this.Parameters.forEach(function(e){
					if(e.CommodityName == $(a.currentTarget).children().find('h5:first-child').text()){
						this.$store.state.market.currentdetail=e;
					}
				}.bind(this));
				this.$router.push({path: '/orderdetail'});
			},
			getVersion: function(){
				this.$http.post(this.PATH + '/getVersion', {emulateJSON: true}, {
						params: {},
						timeout: 5000
				}).then(function(e) {
					var data = e.body;
					if(data.success == true){
						if(data.code == 1&&data.data.version){
							this.$store.state.version.ios = JSON.parse(data.data.version).iOS.version;
							this.$store.state.version.android = JSON.parse(data.data.version).Android.version;
							var version ={
								ios: JSON.parse(data.data.version).iOS.version,
								android: JSON.parse(data.data.version).Android.version
							}
							localStorage.version = JSON.stringify(version);
						}
					}
				}.bind(this), function() {
					this.$refs.dialog.isShow = true;
					this.msg = '网络不给力，请稍后再试！'
				});
			},
			getHotLine: function(){
				this.$http.post(this.PATH + '/hotline', {emulateJSON: true}, {
					params: {},
					timeout: 5000
				}).then(function(e) {
					var data = e.body;
					if(data.success == true && data.code == 1){
						this.$store.state.account.hotLine = data.data.hotline;
					}
				}.bind(this), function() {
					this.$refs.dialog.isShow = true;
					this.msg = '网络不给力，请稍后再试！'
				});
			}
		},
		mounted: function() {
			//初始化高度
//			$("#datalist .cont").height(window.screen.height - $("#topbar").height() - $("#selectbar").height() - $("#datalist .head").height() - $("#navbar").height());
			//初始化行情
			this.initQuoteClient();
			//取当前版本号
			this.getVersion();
			//获取客服热线
			this.getHotLine();
			this.$store.state.menuType = 2;
		},
		beforeUpdate: function(){
//			this.arr = this.Parameters;
		},
		updated: function(){
			//判断网络
			pro.netIsconnected(function(){    //手机
				this.isconnected = true;
				setTimeout(function(){
					this.isconnected = false;
					this.iconIsconnected = true;
					this.colors = 'red';
				}.bind(this), 2000);
				this.initQuoteClient();
			}.bind(this), function(){
				this.$refs.dialog.isShow = true;
				this.msg = '当前网络不可用，请检查网络'
				this.iconIsconnected = false;
				this.colors = '';
				//刷新页面
				window.location.reload();
			}.bind(this));
			
			var EventUtil = { 
				addHandler: function (element, type, handler) { 
					if (element.addEventListener) { 
						element.addEventListener(type, handler, false); 
					}else if(element.attachEvent) { 
						element.attachEvent("on" + type, handler); 
					}else{ 
						element["on" + type] = handler; 
					} 
				} 
			}; 
			EventUtil.addHandler(window, "online", function () { 
				window.location.reload();
			}); 
			EventUtil.addHandler(window, "offline", function () { 
				//alert("Offline"); 
				console.log('网络已断开');
			}); 
		},
		activated:function(){
			this.$store.state.market.currentNo='';
			this.$store.state.isshow.isklineshow = false;
		}
	}
</script>

<style scoped lang="less">
@import url("../assets/css/base.less");
html,body{
	height: 100%;
}
/*ip6p及以上*/
@media (min-width:411px) {
	#home {
		width: 100%;
		overflow: hidden;
		background: @black;
		.disconnect {
			width: 100%;
			height: 50px;
			overflow: hidden;
			background-color: #fff7cc;
			position: fixed;
			left: 0;
			z-index: 1111;
			bottom: 55px;
			.icon{
				float: left;
				display: inline-block;
				width: 18px;
				height: 18px;
				overflow: hidden;
				background: url(../assets/img/tanhao.png) no-repeat center center;
				background-size: 100% 100%;
				margin: 15px 15px 0 15px;
			}
			span{
				display: inline-block;
				line-height: 52px;
				font-size: @fs16;
				color: @black;
			}
		}
		#refresh {
			width: 50px;
			height: 50px;
			background-color: transparent;
			border: none;
			outline: none;
			background: url('../assets/img/refresh.png') no-repeat 16px 16px;
			background-size: 18px 18px;
			position: fixed;
			z-index: 1000;
			right: 0;
			top: 0;
			&.dynamic{
				background-image: url('../assets/img/refresh.gif');
			}
			&.static{
				background-image: url('../assets/img/refresh.png');
			}
		}
		.icon_connected{
			display: inline-block;
			width: 20px;
			height: 20px;
			background: url(../assets/img/tanhao_03.png) no-repeat center center;
			background-size: 100% 100%;
		    margin: 6px 6px 0 0;
			position: fixed;
			top: 0;
			right: 0;
			z-index: 1000;
		}
		.help{
			width: 50px;
			height: 50px;
			overflow: hidden;
			background: url(../assets/img/help.png) no-repeat 15px 15px;
			background-size: 20px 20px;
			border: none;
			outline: none;
			position: fixed;
			top: 0;
			left: 0;
			z-index: 1000;
		}
		.page_cont{
			width: 100%;
			overflow: hidden;
			padding-top: 50px;
			#selectbar{
				position: fixed;
				top: 50px;
				left: 0;
				z-index: 1111;
				width: 100%;
				height: 45px;
				overflow: hidden;
				padding: 0 15px;
				background: @deepblue;
				border-bottom: 1px solid @black;
				ul{
					li{
						float: left;
						height: 45px;
						line-height: 45px;
						border-bottom: 5px solid @deepblue;
						margin-left: 20px;
						color: #ccd4ff;
						font-size: @fs16;
						&:first-child{
							margin: 0;
						}
						&.current{
							color: @yellow;
							border-color: @yellow;
						}
					}
				}
				
			}
			#datalist {
				width: 100%;
				overflow: hidden;
				background: @deepblue;
				padding: 45px 0 0 0;
				ul{
					width: 100%;
					&.head{
						position: fixed;
						top: 95px;
						left: 0;
						li{
							height: 45px;
							line-height: 45px;
							background: #36394d;
							color: @blue;
						}
					}
					&.cont{
						padding-top: 45px;
					}
					li{
						float: left;
						width: 100%;
						height: 80px;
						overflow: hidden;
						padding: 0 15px;
						background: @deepblue;
						border-bottom: 1px solid @black;
						color: @white;
						font-size: @fs16;
						&.bgred{
							background: #502e38;
						}
						&.bggreen{
							background: #294743;
						}
						li{
							line-height: 60px;
							text-align: right;
							padding: 0;
							border: 0;
							background: none;
							&:nth-child(1){
								width: 34%;
								text-align: left;
								h5{
									line-height: 22px;
									color: @blue;
									&:first-child{
										color: @white;
										margin-top: 10px;
									}
								}
							}
							&:nth-child(2){
								width: 20%;
							}
							&:nth-child(3){
								width: 22%;
							}
							&:nth-child(4){
								width: 22%;
							}
							&:nth-child(5){
								width: 22%;
							}
							&.red{
								color: @red;
							}
							&.green{
								color: @green;
							}
							.icon_sj{
								display: inline-block;
								width: 7px;
								height: 7px;
								overflow: hidden;
								background: url(../assets/img/sanjiao.png) no-repeat center center;
								background-size: 100% 100%;
								margin: 0 0 0 2px;
							}
						}
					}
					.tips{
						width: 100%;
						overflow: hidden;
						background: @black;
						text-align: center;
						padding: 15px;
						color: #fff;
						font-size: @fs14;
					}
					.empty{
						width: 100%;
						height: 50px;
						overflow: hidden;
						background: @black;
					}
				}
			}
		}
	}
}

/*ip6*/
@media (min-width:371px) and (max-width:410px) {
    #home {
		width: 100%;
		overflow: hidden;
		background: @black;
		.disconnect {
			width: 100%;
			height: 50px*@ip6;
			overflow: hidden;
			background-color: #fff7cc;
			position: fixed;
			top: 0;
			left: 0;
			z-index: 1111;
			bottom: 55px*@ip6;
			.icon{
				float: left;
				display: inline-block;
				width: 18px*@ip6;
				height: 18px*@ip6;
				overflow: hidden;
				background: url(../assets/img/tanhao.png) no-repeat center center;
				background-size: 100% 100%;
				margin: 15px*@ip6 15px*@ip6 0 15px*@ip6;
			}
			span{
				display: inline-block;
				line-height: 52px*@ip6;
				font-size: @fs16*@ip6;
				color: @black;
			}
		}
		#refresh {
			width: 50px*@ip6;
			height: 50px*@ip6;
			background-color: transparent;
			border: none;
			outline: none;
			background: url('../assets/img/refresh.png') no-repeat 16px*@ip6 16px*@ip6;
			background-size: 18px*@ip6 18px*@ip6;
			position: fixed;
			z-index: 1000;
			right: 0;
			top: 0;
			&.dynamic{
				background-image: url('../assets/img/refresh.gif');
			}
			&.static{
				background-image: url('../assets/img/refresh.png');
			}
		}
		.icon_connected{
			display: inline-block;
			width: 20px*@ip6;
			height: 20px*@ip6;
			background: url(../assets/img/tanhao_03.png) no-repeat center center;
			background-size: 100% 100%;
		    margin: 6px*@ip6 6px*@ip6 0 0;
			position: fixed;
			top: 0;
			right: 0;
			z-index: 1000;
		}
		.help{
			width: 50px*@ip6;
			height: 50px*@ip6;
			overflow: hidden;
			background: url(../assets/img/help.png) no-repeat 15px 15px;
			background-size: 20px*@ip6 20px*@ip6;
			border: none;
			outline: none;
			position: fixed;
			top: 0;
			left: 0;
			z-index: 1000;
		}
		.page_cont{
			width: 100%;
			overflow: hidden;
			padding-top: 50px*@ip6;
			#selectbar{
				position: fixed;
				top: 50px*@ip6;
				left: 0;
				z-index: 1111;
				width: 100%;
				height: 45px*@ip6;
				overflow: hidden;
				padding: 0 15px*@ip6;
				background: @deepblue;
				border-bottom: 1px solid @black;
				ul{
					li{
						float: left;
						height: 45px*@ip6;
						line-height: 45px*@ip6;
						border-bottom: 5px*@ip6 solid @deepblue;
						margin-left: 20px*@ip6;
						color: #ccd4ff;
						font-size: @fs16*@ip6;
						&:first-child{
							margin: 0;
						}
						&.current{
							color: @yellow;
							border-color: @yellow;
						}
					}
				}
				
			}
			#datalist {
				width: 100%;
				overflow: hidden;
				background: @deepblue;
				padding: 45px*@ip6 0 0 0;
				ul{
					width: 100%;
					&.head{
						position: fixed;
						top: 95px*@ip6;
						left: 0;
						li{
							height: 45px*@ip6;
							line-height: 45px*@ip6;
							background: #36394d;
							color: @blue;
						}
					}
					&.cont{
						padding-top: 45px*@ip6;
					}
					li{
						float: left;
						width: 100%;
						height: 70px*@ip6;
						overflow: hidden;
						padding: 0 15px*@ip6;
						background: @deepblue;
						border-bottom: 1px solid @black;
						color: @white;
						font-size: @fs16*@ip6;
						&.bgred{
							background: #502e38;
						}
						&.bggreen{
							background: #294743;
						}
						li{
							line-height: 60px*@ip6;
							text-align: right;
							padding: 0;
							border: 0;
							background: none; 
							&:nth-child(1){
								width: 34%;
								text-align: left;
								h5{
									line-height: 22px*@ip6;
									color: @blue;
									&:first-child{
										color: @white;
										margin-top: 10px*@ip6;
									}
								}
							}
							&:nth-child(2){
								width: 20%;
							}
							&:nth-child(3){
								width: 22%;
							}
							&:nth-child(4){
								width: 22%;
							}
							&:nth-child(5){
								width: 22%;
							}
							&.red{
								color: @red;
							}
							&.green{
								color: @green;
							}
							.icon_sj{
								display: inline-block;
								width: 7px*@ip6;
								height: 7px*@ip6;
								overflow: hidden;
								background: url(../assets/img/sanjiao.png) no-repeat center center;
								background-size: 100% 100%;
								margin: 0 0 0 2px*@ip6;
							}
						}
					}
					.tips{
						width: 100%;
						overflow: hidden;
						background: @black;
						text-align: center;
						padding: 15px*@ip6;
						color: #fff;
						font-size: @fs14*@ip6;
					}
					.empty{
						width: 100%;
						height: 50px*@ip6;
						overflow: hidden;
						background: @black;
					}
				}
			}
		}
	}
}

/*ip5*/
@media(max-width:370px) {
	#home {
		width: 100%;
		overflow: hidden;
		background: @black;
		.disconnect {
			width: 100%;
			height: 50px*@ip5;
			overflow: hidden;
			background-color: #fff7cc;
			position: fixed;
			top: 0;
			left: 0;
			z-index: 1111;
			bottom: 55px*@ip5;
			.icon{
				float: left;
				display: inline-block;
				width: 18px*@ip5;
				height: 18px*@ip5;
				overflow: hidden;
				background: url(../assets/img/tanhao.png) no-repeat center center;
				background-size: 100% 100%;
				margin: 15px*@ip5 15px*@ip5 0 15px*@ip5;
			}
			span{
				display: inline-block;
				line-height: 52px*@ip5;
				font-size: @fs16*@ip5;
				color: @black;
			}
		}
		#refresh {
			width: 50px*@ip5;
			height: 50px*@ip5;
			background-color: transparent;
			border: none;
			outline: none;
			background: url('../assets/img/refresh.png') no-repeat 16px*@ip5 16px*@ip5;
			background-size: 18px*@ip5 18px*@ip5;
			position: fixed;
			z-index: 1000;
			right: 0;
			top: 0;
			&.dynamic{
				background-image: url('../assets/img/refresh.gif');
			}
			&.static{
				background-image: url('../assets/img/refresh.png');
			}
		}
		.icon_connected{
			display: inline-block;
			width: 20px*@ip5;
			height: 20px*@ip5;
			background: url(../assets/img/tanhao_03.png) no-repeat center center;
			background-size: 100% 100%;
		    margin: 6px*@ip5 6px*@ip5 0 0;
			position: fixed;
			top: 0;
			right: 0;
			z-index: 1000;
		}
		.help{
			width: 50px*@ip5;
			height: 50px*@ip5;
			overflow: hidden;
			background: url(../assets/img/help.png) no-repeat 15px 15px;
			background-size: 20px*@ip5 20px*@ip5;
			border: none;
			outline: none;
			position: fixed;
			top: 0;
			left: 0;
			z-index: 1000;
		}
		.page_cont{
			width: 100%;
			overflow: hidden;
			padding-top: 50px*@ip5;
			#selectbar{
				position: fixed;
				top: 50px*@ip5;
				left: 0;
				z-index: 1111;
				width: 100%;
				height: 45px*@ip5;
				overflow: hidden;
				padding: 0 15px*@ip5;
				background: @deepblue;
				border-bottom: 1px solid @black;
				ul{
					li{
						float: left;
						height: 45px*@ip5;
						line-height: 45px*@ip5;
						border-bottom: 5px*@ip5 solid @deepblue;
						margin-left: 20px*@ip5;
						color: #ccd4ff;
						font-size: @fs16*@ip5;
						&:first-child{
							margin: 0;
						}
						&.current{
							color: @yellow;
							border-color: @yellow;
						}
					}
				}
				
			}
			#datalist {
				width: 100%;
				overflow: hidden;
				background: @deepblue;
				padding: 45px*@ip5 0 0 0;
				ul{
					width: 100%;
					&.head{
						position: fixed;
						top: 95px*@ip5;
						left: 0;
						li{
							height: 45px*@ip5;
							line-height: 45px*@ip5;
							background: #36394d;
							color: @blue;
						}
					}
					&.cont{
						padding-top: 45px*@ip5;
					}
					li{
						float: left;
						width: 100%;
						height: 75px*@ip5;
						overflow: hidden;
						padding: 0 15px*@ip5;
						background: @deepblue;
						border-bottom: 1px solid @black;
						color: @white;
						font-size: @fs16*@ip5;
						&.bgred{
							background: #502e38;
						}
						&.bggreen{
							background: #294743;
						}
						li{
							line-height: 60px*@ip5;
							text-align: right;
							padding: 0;
							border: 0; 
							background: none; 
							&:nth-child(1){
								width: 34%;
								text-align: left;
								h5{
									line-height: 22px*@ip5;
									color: @blue;
									&:first-child{
										color: @white;
										margin-top: 10px*@ip5;
									}
								}
							}
							&:nth-child(2){
								width: 20%;
							}
							&:nth-child(3){
								width: 22%;
							}
							&:nth-child(4){
								width: 22%;
							}
							&:nth-child(5){
								width: 22%;
							}
							&.red{
								color: @red;
							}
							&.green{
								color: @green;
							}
							.icon_sj{
								display: inline-block;
								width: 7px*@ip5;
								height: 7px*@ip5;
								overflow: hidden;
								background: url(../assets/img/sanjiao.png) no-repeat center center;
								background-size: 100% 100%;
								margin: 0 0 0 2px*@ip5;
							}
						}
					}
					.tips{
						width: 100%;
						overflow: hidden;
						background: @black;
						text-align: center;
						padding: 15px*@ip5;
						color: #fff;
						font-size: @fs14*@ip5;
					}
					.empty{
						width: 100%;
						height: 50px*@ip5;
						overflow: hidden;
						background: @black;
					}
				}
			}
		}
	}
}		
</style>