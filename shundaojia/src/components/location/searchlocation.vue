<template>
	<div class="searchlocation">
		<!-- 头部 -->
		<div class="header">
			<div class="header_back" @click="back()">
				<img src="../../common/image/fanhui.png" width="9" height="19"/>
			</div>
			<div class="header_add">
				选择收货地址
			</div>
			<div class="header_address" @click="newaddress()">
				新增地址
			</div>
		</div>
		<!-- 输入框 -->
		<div class="inp">
			<div class="inp_r" @click="city()">
				<span class="ipn_address">{{cityname}}</span>
				<img src="./xiaosanjiao.png" />
			</div>
			<!-- 搜索框 -->
			<div class="search">
				<!--<span class="search_pic">-->
     				<!--<span class="sdj-hshouyesousuo"></span>-->
     				<img src="../../common/image/shopsearch.png" width="16" height="16" style="margin-right: 8px;margin-left: 12px;"/>
				<!--</span>-->
				<input v-model="query" @focus="focus()" class="search_inp" placeholder="请输入地址">
				<img @click="clear()" v-show="textinput" src="../../common/image/shanchu.png" width="16" height="16" style="margin-right: 4px;"/>
				<!--<span @click="clear()" v-show="textinput" class="sdj-logdele"><span class="path1"></span><span class="path2"></span><span class="path3"></span></span>-->
			</div>
			<!-- 搜索按钮 -->
			<div v-show="textinput" class="search_btn" @click="search()">取消</div>
		</div>
		<!--正常页面-->
		<div class="content" v-show="!cityhidden">

			<!-- 当前位置 -->
			<div v-show="home">

				<div class="current">
					当前位置
				</div>
				<!-- 重新定位 -->
				<div class="afresh">
					<span @click="cxdw()" class="afresh_loca">{{location}}</span>
					<div class="reposition" @click="chongxin()">
						
						<!--<span class="sdj-chongxindingwei"></span>-->
						<img style="vertical-align: middle;" src="../../common/image/chongxindingwei.png" width="18" height="18"/>
						<span class="reposition_z">重新定位</span>
					</div>
				</div>
				<!-- 收货地址 -->
				<div class="current">
					收货地址
				</div>
				<!-- 收货地址内容 -->
				<scroll :data="mineAddress" class="scroll">
					<div id="">

						<!-- 家 -->
						<div class="goods" v-for="(val,index) in mineAddress" @click="mineAddressClick(val)">
							<div class="goods_icon">
								<button v-show="val.tag" :class="iconName(val.tag)">{{val.tag}}</button>
							</div>
							<div class="goods_content">
								<div class="goods_loca">
									{{val.locaddress}}{{val.address}}
								</div>
								<div class="goods_peo">
									<span>{{val.name}}</span>
									<span>{{val.mobile}}</span>
								</div>
							</div>
						</div>
						
					</div>
				</scroll>

			</div>

			<!-- 历史搜索 -->
			<div v-if="searchhistory.length>0" v-show="hissearch">
				<div class="his">
					<span class="his_word">历史搜索</span>
					<img src="../../common/image/qingkong.png" width="15" height="16" @click="showToggle()"/>
					<!--<span class="sdj-addshanchu" ></span>-->
				</div>
				<div class="history">
					<div class="history_z" v-for="val in searchhistory" @click="historyClick(val)">
						{{val.location}}
					</div> 

				</div>
			</div>

			<!-- 搜索结果 -->
			<scroll :data="name.tips" ref="search" v-show="name&&this.query" class="scroll2">
				<div class="results">
					<div class="results_z" v-show="val.location" v-for="(val,index) in name.tips" @click="select(val)">
						<div class="results_t">
							{{val.name}}
						</div>
						<div class="results_b">
							{{val.address}}
						</div>
					</div>

				</div>
				<!-- 没有搜索结果 -->
				<div class="results_no" v-show="!name.tips">
					<div class="search_no">
						<div class="no_icon"><img src="../../common/image/cha7.png"/></div>
						<div class="no_word1">没有搜索结果</div>
						<div class="no_word2">换个关键词试试</div>
					</div>
				</div>
			</scroll>

			<!-- 清除历史记录的弹框 -->
			<div class="clear" v-show="isShow">
				<div class="bounces">
					<div class="clear_record">
						清除历史记录?
					</div>

					<div class="clear_word">
						<div class="cancel" @click="isShow=false">取消</div>
						<div class="confirm" @click="historyt()">确认</div>
					</div>
				</div>
			</div>

		</div>
		<!--列表块-->
		<div class="" v-if="cityhidden">
			<div class="current">
				当前定位城市
			</div>
			<!-- 重新定位 -->
			<div class="afresh">
				<span class="afresh_loca">{{cityname}}</span>
				<div class="reposition" @click="chongxin()">
					<span class="sdj-chongxindingwei"></span>
					<span class="reposition_z">重新定位</span>
				</div>
			</div>
			<!--列表-->
			<div class="citylist">
				<listview :data="citylist" @select="selectcity"></listview>
			</div>
		</div>

		<toptip ref="toptip">
			<div class="toptip">
				{{message}}
			</div>
		</toptip>
	</div>

</template>

<script>
	//引入黑色弹窗
	import toptip from '../../base/top-tip/top-tip.vue'
	//取回本地定位地址
	import { loadLocation, saveLocation, saveSearch, loadSearch, clearSearch } from '../../common/js/catch.js'
	//引入滑动组件
	import Scroll from '../../base/scroll/scroll.vue'
	// 引入高德地图
	import AMap from 'AMap'
	//引入列表组件
	import listview from '../../base/listview/listview.vue'
	//引入城市
	import { cityData} from '../../common/js/city.js'
	import { loadMine} from '../../common/js/catch.js'
	
	import { Api } from '../../common/js/api.js'
	export default {
		data() {
			return {
				mine:{},
				mineAddress:[],
				
				
				scrolldata: [1, 2, 3, 4, 5],//收货地址
				citylist: [], //城市列表
				cityname: '北京市',//城市名称
				name: {}, //搜索列表数据
				searchhistory: [],//搜索历史
				message: '', //弹框内容
				query: '',
				//控制显示隐藏字段
				cityhidden: false, //控制城市列表
				dangqian: false,//控制历史记录
				textinput: false, //控制搜索按钮
				isShow: false,//控制清空历史记录弹框
//				shanchu: true,

				geocoder: {},//坐标地址对象
				lat: '', //经度
				lon: '', //纬度
				location: '重新定位', //地址
				autoOptions: {
					city: "", //城市，默认全国
					// input:"input_id"
				},
				autocomplete: '',
				localTiaoZhuan:true,//控制重新定位跳转首页
			}
		},
		props: {

		},
		methods: {
			mineAddressClick(val){
				let location = {};
				location.location = val.locaddress
					location.lat = val.latitude //经度
					location.lon = val.longtitude //纬度
				saveLocation(location)
				
				this.$router.push('/homepage')
			},
			//控制地址列表样式
			iconName(tag){
				if(tag==='家'){
					return 'icon_home'
				}else if(tag==='公司'){
					return 'icon_company'
				}else if(tag==='学校'){
					return 'icon_school'
				}else{
					return 'goods_icon'
				}
			},
			showToggle: function() {
				this.isShow = !this.isShow;
			},
			//重新定位
			chongxin() {
				this.location = '正在定位'
				this.relocation()
			},
			//清空
			clear() {
				this.query = '',
					this.name = {}

				this.dangqian = false //控制历史记录
				this.textinput = false
			},
			//清空搜索历史
			historyt() {
				this.isShow = false
//				this.shanchu = false
				clearSearch()
				this.searchhistory = []
			},
			//输入框获取焦点
			focus() {
				this.cityhidden = false
				this.textinput = true
				this.dangqian = true
			},
			//跳转到新增地址
			newaddress() {
				this.$router.push('/newaddress')
			},
			//返回
			back() {
				this.$router.push('/homepage')
			},
			//控制城市列表
			city() {
				this.citylist = []
				//城市列表
				this.citylist = cityData
				this.cityhidden = !this.cityhidden
			},
			//选中城市
			selectcity(item) {
				this.cityname = item.name
				this.cityhidden = !this.cityhidden
			},
			//选中搜索历史
			historyClick(val){
				
				this.query=val.location
			},
			//重新定位跳转首页
			cxdw() {
				if(this.localTiaoZhuan) {
					let location = {};
					location.location = this.location
					location.lat = this.lat //经度
					location.lon = this.lng //纬度

					saveLocation(location)
					this.$router.push('/homepage')
				} else {
					this.chongxin()
				}

			},

			//选中搜索列表，跳转首页
			select(val) {
				let location = {};
				location.location = val.name
				if(val.location) {
					location.lat = val.location.lat //经度
					location.lon = val.location.lng //纬度
					saveSearch(location)
				}
				saveLocation(location)
				
				this.$router.push('/homepage')

			},
			//获取搜索列表
			search() {
				this.query = '',
				this.name = {}

				this.dangqian = false //控制历史记录
				this.textinput = false

			},
			//高德搜索列表
			searchlist() {
				this.autocomplete.search(this.query, (status, result) => {
					this.name = result
					
				});
			},
			// 重新定位
			relocation() {
				
				let geolocation = new AMap.Geolocation({
					enableHighAccuracy: true, //是否使用高精度定位，默认:true
					timeout: 10000, //超过10秒后停止定位，默认：无穷大
					maximumAge: 0, //定位结果缓存0毫秒，默认：0
					convert: true, //自动偏移坐标，偏移后的坐标为高德坐标，默认：true
					showButton: true, //显示定位按钮，默认：true
					buttonPosition: 'LB', //定位按钮停靠位置，默认：'LB'，左下角
					buttonOffset: new AMap.Pixel(10, 20), //定位按钮与设置的停靠位置的偏移量，默认：Pixel(10, 20)
					showMarker: true, //定位成功后在定位到的位置显示点标记，默认：true
					showCircle: true, //定位成功后用圆圈表示定位精度范围，默认：true
					panToLocation: true, //定位成功后将定位到的位置作为地图中心点，默认：true
					zoomToAccuracy: true //定位成功后调整地图视野范围使定位位置及精度范围视野内可见，默认：false
				});
				geolocation.getCurrentPosition();
				AMap.event.addListener(geolocation, 'complete', this.onComplete); //返回定位信息
				AMap.event.addListener(geolocation, 'error', this.onError); //返回定位出错信息
			},
			// 定位成功返回的地址
			onComplete(data) {
				
				this.location = data.formattedAddress
				this.lat = data.position.lat
				this.lon = data.position.lng
				this.localTiaoZhuan=true
				this.getAddress([data.position.lng,data.position.lat])
			},
			// 定位失败返回的地址
			onError(data) {
				this.message = "定位失败"
				this.$refs.toptip.show()
				this.location = "定位失败"
				this.localTiaoZhuan=false
			},

			// 坐标转地址
			getAddress(address) {
				this.geocoder = new AMap.Geocoder({
					city: "010" //城市，默认：“全国”
				});
				this.geocoder.getAddress(address, function(status, result) {
					if(status === 'complete' && result.info === 'OK') {
						if(!result.regeocode.addressComponent.city) {
							this.cityname = result.regeocode.addressComponent.province
						} else {
							this.cityname = result.regeocode.addressComponent.city
						}

					} else {
						this.message = "定位失败"
						this.$refs.toptip.show()
					}
				}.bind(this));
			},
			// 地址转坐标
			getLocation(address) {
				this.geocoder = new AMap.Geocoder({
					city: "010" //城市，默认：“全国”
				});
				this.geocoder.getLocation(address, function(status, result) {
					if(status === 'complete' && result.info === 'OK') {
						

					} else {
						this.message = "定位失败"
						this.$refs.toptip.show()
					}
				}.bind(this));
			},
			//地址列表
			_list_() {
				this.$ajax.post(Api(), {
						'i': 2,
						'actionId': '102006',
						'mid': this.mine.memberid,
						'token': this.mine.token,
						

					})
					.then((response) => {
						if(response.data.code == 1) {
							
							this.mineAddress=response.data.data
						} else {
							this.message = response.data.msg
							this.$refs.toptip.show()
						}
					})
					.catch((error) => {
						this.message = '请求错误'
						this.$refs.toptip.show()
					});
			}
		},
		computed: {
			//历史搜索的显示和隐藏
			hissearch() {
				if(!this.cityhidden && !this.name.tips && !this.query) {
					return true
				} else {
					return false
				}
			},
			home() {
				if(!this.cityhidden && this.dangqian) {
					return false
				} else {
					return true
				}
			}
		},
		components: {
			Scroll,
			listview,
			toptip
		},
		mounted() {

		},
		created() {
			this.mine=loadMine()
			this._list_()
			//搜索列表
			this.autocomplete = new AMap.Autocomplete(this.autoOptions)

			//搜索历史
				let loadSearchList=[]
			loadSearch().forEach((item)=>{
				if(item){
					loadSearchList.push(item)
				}
			})
			this.searchhistory=loadSearchList
			//定位地址
			let location = loadLocation()
			
			if(location.lat) {
				
				this.location = location.location
				this.lat = location.lat //经度
				this.lng = location.lon //纬度
				
				this.getAddress([this.lng, this.lat])
			} else {
				//定位
				this.localTiaoZhuan=false
				this.relocation()
			}

		},
		watch: {
			query(newQuery) {
				this.searchlist()
				if(!newQuery) {
					setTimeout(() => {
						this.$refs.search.refresh()
					}, 20)
				}
			}
		}
	}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
	.searchlocation {
		position: fixed;
		top: 0;
		bottom: 0;
		left: 0;
		right: 0;
		background: #FFFFFF;
		z-index: 1000;
		box-sizing: border-box;
		background: #F5F5F5;
	}
	
	input {
		outline: none;
		border: none;
	}
	
	button {
		outline: none;
		border: 0;
	}
	/*滚动样式*/
	
	.scroll {
		position: fixed;
		top: 224px;
		bottom: 0px;
		left: 0px;
		right: 0px;
		z-index: 2000;
		background: #FFFFFF;
		overflow: hidden;
	}
	
	.scroll2 {
		position: fixed;
		top: 104px;
		bottom: 0px;
		left: 0px;
		right: 0px;
		z-index: 2000;
		background: #FFFFFF;
		overflow: hidden;
	}
	/*滚动样式结束*/
	/*头部*/
	
	.header {
		position: fixed;
		width: 100%;
		height: 44px;
		line-height: 44px;
		display: flex;
		background: #fff;
		padding: 0px 12px;
		box-sizing: border-box;
	}
	
	.header_back {
		width: 70px;
	}
	
	.header_add {
		flex: 1;
		text-align: center;
		color: #4C4C4C;
		letter-spacing: 0;
		font-family: PingFangSC-Medium;
		font-size: 18px;
		font-weight: 600;
	}
	
	.header_address {
		width: 70px;
		text-align: right;
		font-family: PingFangSC-Medium;
		font-size: 15px;
		color: #4C4C4C;
		letter-spacing: 0;
		font-weight: 600;
	}
	/*输入框*/
	
	.inp {
		background: #fff;
		height: 48px;
		align-items: center;
		margin-top: 44px;
		display: flex;
		width: 100%;
		padding: 0px 12px;
		box-sizing: border-box;
	}
	
	.inp_r {
		width: 69px;
		overflow: hidden;
		/*超出文档显示省略号*/
		text-overflow: ellipsis;
		white-space: nowrap;
		display: flex;
		align-items: center;
	}
	
	.ipn_address {
		display: inline-block;
		font-size: 15px;
		color: #4C4C4C;
		width: 45px;
		overflow: hidden;
		/*超出文档显示省略号*/
		text-overflow: ellipsis;
		white-space: nowrap;
	}
	
	.inp_r img {
		width: 8px;
		height: 5px;
		vertical-align: middle;
	}
	/*搜索框*/
	
	.search {
		height: 32px;
		flex: 1;
		background: #EDEDED;
		line-height: 48px;
		display: flex;
		width: 0;
		/*padding-right: 12px;*/
		align-items: center;
	}
	
	.sdj-logdele {
		line-height: 32px;
		padding-right: 4px;
	}
	
	.search_pic {
		padding: 0px 8px;
		height: 32px;
		line-height: 32px;
		/*text-align: center;*/
	}
	
	.search_inp {
		background: #EDEDED;
		padding-right: 3px;
		font-family: PingFangSC-Medium;
		font-size: 15px;
		color: #333333;
		flex: 1;
		width: 0;
	}
	/*搜索按钮*/
	
	.search_btn {
		width: 52px;
		height: 32px;
		background: #FF884C;
		font-family: PingFangSC-Medium;
		font-size: 14px;
		color: #FFFFFF;
		line-height: 32px;
		text-align: center;
		margin-left: 12px;
	}
	/*当前位置*/
	
	.current {
		height: 42px;
		line-height: 42px;
		padding-left: 12px;
		font-family: PingFangSC-Medium;
		font-size: 14px;
		color: #999999;
	}
	/*重新定位*/
	
	.afresh {
		height: 48px;
		align-items: center;
		display: flex;
		padding: 0px 12px;
		background: #fff;
	}
	
	.afresh_loca {
		font-family: PingFangSC-Medium;
		font-size: 15px;
		color: #4C4C4C;
		line-height: 20px;
		flex: 1;
		overflow: hidden;
		/*超出文档显示省略号*/
		text-overflow: ellipsis;
		white-space: nowrap;
	}
	
	.reposition {
		text-align: right;
		width: 95px;
	}
	
	.sdj-chongxindingwei {
		vertical-align: middle;
	}
	
	.reposition_z {
		font-family: PingFangSC-Medium;
		font-size: 14px;
		color: #FF884C;
		line-height: 18px;
		padding-left: 5px;
	}
	/*收货地址内容*/
	
	.goods {
		display: flex;
		background: #fff;
		padding: 16px 0px 16px 12px;
		border-bottom: 1px solid #eee;
	}
	/*家*/
	
	.goods_icon {
		width: 38px;
	}
	
	.icon_home {
		width: 38px;
		height: 18px;
		background: rgba(255, 136, 76, 0.10);
		border-radius: 2px;
		font-family: PingFangSC-Medium;
		font-size: 12px;
		color: #FF884C;
		line-height: 16px;
	}
	
	.goods_content {
		flex: 1;
		padding-left: 8px;
		padding-right: 12px;
	}
	
	.goods_loca {
		font-family: PingFangSC-Medium;
		font-size: 15px;
		color: #4C4C4C;
		line-height: 20px;
	}
	
	.goods_peo {
		padding-top: 4px;
		font-family: PingFangSC-Regular;
		font-size: 14px;
		color: #999999;
		line-height: 18px;
	}
	
	.goods_peo span {
		margin-right: 24px;
	}
	/*公司*/
	
	.icon_company {
		width: 38px;
		height: 18px;
		background: rgba(76, 166, 255, 0.10);
		border-radius: 2px;
		font-family: PingFangSC-Medium;
		font-size: 12px;
		color: #4CA6FF;
		line-height: 16px;
	}
	/*学校*/
	
	.icon_school {
		width: 38px;
		height: 18px;
		background: rgba(255, 76, 121, 0.10);
		border-radius: 2px;
		font-family: PingFangSC-Medium;
		font-size: 12px;
		color: #FF4C79;
		line-height: 16px;
	}
	/*历史搜索*/
	
	.his {
		height: 42px;
		align-items: center;
		padding: 0px 12px;
		font-family: PingFangSC-Medium;
		font-size: 14px;
		color: #999999;
		display: flex;
		width: 100%;
		box-sizing: border-box;
	}
	
	.his_word {
		flex: 1;
	}
	
	.sdj-addshanchu {
		text-align: right;
		/*display: block;*/
	}
	
	.history {
		padding-left: 12px;
		background: #fff;
	}
	
	.history_z {
		border-bottom: 1px solid #eee;
		font-family: PingFangSC-Medium;
		font-size: 15px;
		color: #4C4C4C;
		line-height: 20px;
		height: 52px;
		line-height: 52px;
	}
	/*搜索结果*/
	
	.results {
		padding-left: 12px;
		background: #fff;
		margin-top: 12px;
	}
	
	.results_z {
		padding: 16px 12px 16px 0px;
		border-bottom: 1px solid #eee;
	}
	
	.results_t {
		font-family: PingFangSC-Medium;
		font-size: 15px;
		color: #4C4C4C;
		line-height: 20px;
		font-weight: 600;
	}
	
	.results_b {
		margin-top: 4px;
		font-family: PingFangSC-Regular;
		font-size: 13px;
		color: #666666;
		line-height: 18px;
	}
	/*没有搜索结果*/
	
	.results_no {
		display: flex;
		width: 100%;
		height: 100%;
		align-items: center;
		justify-content: center;
	}
	
	.search_no {
		margin-top: -160px;
	}
	
	.no_icon {
		width: 156px;
		height: 153px;
		/*background: #ccc;*/
	}
	.no_icon img{
		width: 100%;
		height: 100%;
	}
	
	.no_word1 {
		font-family: PingFangSC-Medium;
		font-size: 16px;
		color: #999999;
		text-align: center;
		margin-top: 14px;
	}
	
	.no_word2 {
		font-family: PingFangSC-Regular;
		font-size: 13px;
		color: #999999;
		text-align: center;
		margin-top: 4px;
	}
	/*清除历史记录的弹框*/
	
	.clear {
		position: fixed;
		top: 0;
		bottom: 0;
		left: 0;
		right: 0;
		background: #FFFFFF;
		z-index: 1;
		box-sizing: border-box;
		background: rgba(0, 0, 0, 0.5);
	}
	
	.bounces {
		position: fixed;
		width: 270px;
		height: 112px;
		background: #FFFFFF;
		border-radius: 12px;
		z-index: 100;
		left: 50%;
		top: 50%;
		margin-left: -135px;
		margin-top: -56px;
	}
	
	.clear_record {
		font-family: PingFangSC-Medium;
		font-size: 16px;
		color: #4C4C4C;
		letter-spacing: 0;
		height: 68px;
		line-height: 68px;
		text-align: center;
		border-bottom: 1px solid #EDEDED;
	}
	
	.clear_word {
		height: 43px;
		line-height: 43px;
		font-family: PingFangSC-Medium;
		font-size: 16px;
		color: #4CA6FF;
		letter-spacing: 0;
	}
	
	.cancel {
		width: 135px;
		border-right: 1px solid #EDEDED;
		float: left;
		text-align: center;
	}
	
	.confirm {
		float: right;
		text-align: center;
		width: 134px;
	}
	/*城市列表*/
	
	.citylist {
		position: fixed;
		top: 182px;
		bottom: 0;
		left: 0px;
		right: 0px;
	}
	/*弹窗*/
	
	.toptip {
		display: inline-block;
		padding: 24px;
		background: #666;
		color: #FFFFFF;
		border-radius: 8px;
		box-sizing: border-box;
		font-size: 16px;
		color: #FFFFFF;
		letter-spacing: 0;
		line-height: 18px;
		
	}
</style>