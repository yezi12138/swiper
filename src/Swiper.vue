<template>
	<div class="swiper" :style="{height:height}">
		<div class="swiper-items-wrap" ref="wrap">
			<slot></slot>
		</div>
		<div class="swiper-indicators-wrap" :class="{'indicatorType-1':indicatorType===1,'indicatorType-2':indicatorType===2}">
			<div class="swiper-indicator" :class="{'is-active':$index==index}" v-for="(page,$index) in pages" v-show="showIndicators" @click="nevigateTo($event,$index)"></div>
		</div>
	</div>
</template>

<script>
	export default{
		data(){
			return {
				isReady:false,
				pages:[],
				index:0,
				animating: false,
				dragState:{},
				timer:null
			}
		},
		props: {
			speed: {
				type: Number,
				default: 3000
			},

			loop: {
				type: Boolean,
				default: true
			},

			showIndicators: {
				type: Boolean,
				default: true
			},

			indicatorType: {
				type: Number,
				default: 1
			},

			height: {
				type:String,
				default: '250px'
			}
		},
		mounted(){
			this.$nextTick(function(){

				this.initPages();
				if(this.pages.length<=1)return;
				if(this.loop){
					this.autoPlay();
				}
				this.$el.addEventListener('touchstart', (event) => {
					this.doOnTouchStart(event);
				});
				this.$el.addEventListener('touchmove', (event) => {
					this.doOnTouchMove(event);
				});
				this.$el.addEventListener('touchend', (event) => {
					this.doOnTouchEnd(event);
				});
			})
		},
		methods:{
			initPages(){ 
				//初始化
				var wrap = this.$refs.wrap;
				var children = wrap.querySelectorAll('.swiper-item');
				var pages = [];
				var width = this.$el.clientWidth;
				children.forEach(function(child,index){
					pages.push(child);
				})
				this.pages = pages;
				this.dragState.pageWidth = width;
			},
			doOnTouchStart(event){
				var dragState = this.dragState;
				var touch = event.touches[0];
				var date = new Date();
				dragState.startTime = date.getSeconds()*60 + date.getMilliseconds();
				dragState.startLeft = touch.pageX;
				//取消之前存好的方向
				dragState.toward = '';
				//模板化元素，并且初始化位置
				this.initPos();
				//清楚定时器
				this.clearTimer();

			},
			doOnTouchMove(event){

				var dragState = this.dragState;
				var touch = event.touches[0];
				dragState.currentLeft = touch.pageX;
				var offsetLeft = dragState.currentLeft - dragState.startLeft;
				//防止活动距离超出页面宽度
				offsetLeft = Math.min(Math.max(-dragState.pageWidth, offsetLeft), dragState.pageWidth);
				var toward = offsetLeft>0?'prev':'next';
				dragState.offsetLeft = offsetLeft;
				dragState.toward = toward;
				//页面移动
				this.translate(dragState.currentPage,offsetLeft);
				if(toward === 'prev'){
					this.translate(dragState.prevPage,-dragState.pageWidth+offsetLeft);
				}
				if(toward === 'next'){
					this.translate(dragState.nextPage,offsetLeft+dragState.pageWidth);
				}
			},
			doOnTouchEnd(event){

				var dragState = this.dragState;
				var date = new Date();
				dragState.endTime = date.getSeconds()*60 + date.getMilliseconds();
				var duration = dragState.endTime - dragState.startTime;
				/*if(duration<=300){
					console.log("click");
					return;
				}*/
				if(Math.abs(dragState.offsetLeft)<dragState.pageWidth/2){
					console.log("撤销动作");
					this.initPos();
				}else if(dragState.toward === 'next'){
					this.destoryBlock();
					this.index++;
					this.initPos();
				}else if(dragState.toward === 'prev'){
					this.destoryBlock();
					this.index--;
					this.initPos();
				}
				if(this.loop){
					this.autoPlay();
				}
			},
			translate(element,offset,speed,callback){
				var speed = speed || 300;
				if(this.pages.length == 2){
					element.style.webkitTransition = '';
					element.style.webkitTransform = `translate3d(${offset}px, 0, 0)`;
				}else{
					element.style.webkitTransition = '-webkit-transform ' + speed + 'ms linear';
					element.style.webkitTransform = `translate3d(${offset}px, 0, 0)`;
				}
				
				if (callback) {
					callback.apply(this, arguments);
				}
			},
			initPos(){
				var dragState = this.dragState;
				var index = this.index;
				var children = this.pages;
				//判断index的位置
				if(index>children.length-1){
					index = 0;
				}else if(index<0){
					index = children.length-1;
				}
				this.index = index;
				var oldPrevPage = dragState.prevPage;
				var oldNextPage = dragState.nextPage;
				dragState.prevPage = children[index-1]?children[index-1]:children[children.length-1];
				dragState.currentPage = children[index];
				dragState.nextPage = children[index+1]?children[index+1]:children[0];

				//修正只有3个页面时候切换的bug
				if(oldPrevPage === dragState.nextPage){
					dragState.nextPage.style.webkitTransition = '';
				}
				if(oldNextPage === dragState.prevPage){
					 dragState.prevPage.style.webkitTransition = '';
				}
				//页面初始化
				if(dragState.prevPage){
					dragState.prevPage.style.display = 'block';
					dragState.prevPage.style.transform = `translateX(-${dragState.pageWidth}px)`;
				}
				if(dragState.currentPage){
					dragState.currentPage.style.display = 'block';
					dragState.currentPage.style.transform = `translateX(0)`;
				}
				if(dragState.nextPage){
					dragState.nextPage.style.display = 'block';
					dragState.nextPage.style.transform = `translateX(${dragState.pageWidth}px)`;
				}
			},
			destoryBlock(){
				var dragState = this.dragState;
				dragState.prevPage.style.display = 'none';
				dragState.prevPage.style.webkitTransform = '';
				dragState.currentPage.style.display = 'none';
				dragState.currentPage.style.webkitTransform ='';
				dragState.nextPage.style.display = 'none';
				dragState.nextPage.style.webkitTransform = '';
			},
			autoPlay(){
				if(!this.loop) return;
				this.timer = setInterval(() => {
					this.doAnimate('next');	
				}, this.speed);
			},
			doAnimate(toward){
				var toward = toward || 'next';
				if(toward === 'next'){
					var dragState = this.dragState;
					this.initPos();
					this.translate(dragState.currentPage,-dragState.pageWidth);
					this.translate(dragState.nextPage,0);
					this.destoryBlock();
					this.index++;
					this.initPos();	
				}
			},
			clearTimer(){
				clearInterval(this.timer);
			},
			nevigateTo(e,index){
				this.index = index;
				this.destoryBlock();
				this.initPos();
			}
		}

	}
</script>

<style>
	.swiper{
		position: relative;
		overflow: hidden;
		height:250px;
	}
	.swiper-items-wrap{
		position: relative;
		height: 100%;
		font-size:0;
		overflow: hidden;
	}
	.swiper-items-wrap>.swiper-item{
		position: absolute;
		top:0;
		left:0;
		width:100%;
		height:100%;
		display:none;
		background: #999;
		border:0;
	}
	.swiper-items-wrap>.swiper-item img{
		width:100%;
		height:100%;
	}
	.swiper-items-wrap div:first-child{
		display:block;
	}

	.swiper-indicators-wrap{
		position: absolute;
		bottom:10px;
		z-index: 1000;
	}
	.indicatorType-1{
		left:50%;
		transform: translate(-50%);
	}
	.indicatorType-2{
		right:30px;
	}
	.indicatorType-2 .swiper-indicator{
		display:inline-block;
		width:8px;
		height:8px;
		border-radius:100%;
		background-color: #999;
		opacity:0.3;
		margin-right:8px;
	}
	.indicatorType-1 .swiper-indicator{
		display:inline-block;
		width:13px;
		height:13px;
		border-radius:100%;
		background-color: #999;
		opacity:0.3;
		margin-right:8px;
	}
	.indicatorType-1 .is-active,.indicatorType-2 .is-active{
		background-color: #fff;
		opacity:1;
	}
	.swiper-indicator:last-child{
		margin-right:0;
	}
</style>