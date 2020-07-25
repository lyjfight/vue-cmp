<template>
	<div class="dragValite">
		<div class="dragValite-panel animated" style="display: none;" :class="{fadeInUp: isPanelActive == '1', fadeInDown: isPanelActive == '2'}" v-if="puzzle">
			<div class="dragValite-placeholder">
				<div class="dragValite_bgimg">
					<img class="dragValite_bg-img" style="border-radius: 2px" :src="imgData.bgImg">
					<img class="dragValite_lyjmoveimg" :src="imgData.moveImg">
				</div>
			</div>
		</div>
		<div class="dragValite-control">
			<div class="dragValite_slide_indicator" :class="{__move: isMove, __success: isSuccess, __fail: isFail}" :style="{width: indicatorWidth + 'px'}"></div>
			<div class="dragValite_slider" :class="{__success: isSuccess, __fail: isFail}" :style="{left: sliderLeft + 'px'}">
				<span class="dragValite_slider__icon"></span>
			</div>
			<div class="dragValite_tip">
				<span class="dragValite_tip-text" v-show="isShowTip && !isSuccess && !isFail">
					向右拖动滑块填充拼图
				</span>
			</div>
		</div>
	</div>
</template>
<script>
class seqDrag {
	constructor() {

	}
	static init({
			down: downCallBack,
			move: moveCallBack,
			up: upCallBack,
			panel: panelCallBack
		}, range) {
		seqDrag.panel=document.querySelector('.dragValite-panel');
		seqDrag.moveImg=document.querySelector('.dragValite_lyjmoveimg')
		seqDrag.bgImg=document.querySelector('.dragValite_bgimg')
		seqDrag.slider=document.querySelector('.dragValite_slider');
		seqDrag.control=document.querySelector(".dragValite-control");
		seqDrag.indicator=document.querySelector(".dragValite_slide_indicator");

		// seqDrag.slider.style.left = seqDrag.slider.offsetLeft + 'px'

		seqDrag.panel && seqDrag.control.addEventListener('mouseenter', seqDrag.panelHandler)
		seqDrag.panel && seqDrag.control.addEventListener('mouseleave', seqDrag.panelHandler)
		seqDrag.slider.addEventListener('mousedown',seqDrag.drag);
		document.addEventListener('mousemove',seqDrag.drag);
		document.addEventListener('mouseup',seqDrag.drag); 
		seqDrag.toggle=false; //是否点中元素
		seqDrag.x1=NaN
		seqDrag.ele=null
		seqDrag.startX=NaN
		seqDrag.maxLeft=seqDrag.control.offsetWidth - seqDrag.slider.offsetWidth
		seqDrag.downCallBack = downCallBack
		seqDrag.moveCallBack = moveCallBack
		seqDrag.upCallBack = upCallBack
		seqDrag.panelCallBack = panelCallBack
		seqDrag.range = range
		seqDrag.fail = false
	}
	static reset(range) {
		seqDrag.range = range
	}
	static close() {
		seqDrag.panel && seqDrag.control && seqDrag.control.removeEventListener('mouseenter', seqDrag.panelHandler)
		seqDrag.panel && seqDrag.control && seqDrag.control.removeEventListener('mouseleave', seqDrag.panelHandler)
		seqDrag.slider && seqDrag.slider.removeEventListener('mousedown',seqDrag.drag);
		document.removeEventListener('mousemove',seqDrag.drag);
		document.removeEventListener('mouseup',seqDrag.drag);
	}
	static drag(ev){
		var ev=ev||window.event; //鼠标事件
		ev.preventDefault(); //阻止默认事件
		switch(ev.type){
			case 'mousedown':
				if(ev.target.tagName==='DIV'){
					seqDrag.ele=ev.target;
					seqDrag.toggle=true;
					seqDrag.x1=ev.clientX;
					seqDrag.startX=seqDrag.ele.offsetLeft;
					seqDrag.downCallBack && seqDrag.downCallBack()
				}
				break;
			case 'mousemove':
				if(seqDrag.toggle){
					var x2=ev.clientX,
							nowX=seqDrag.startX+x2-seqDrag.x1
					if(seqDrag.panel) {
						seqDrag.imgMaxLeft=seqDrag.bgImg.offsetWidth - seqDrag.moveImg.offsetWidth
					}
					if(nowX <= 0) {
						seqDrag.ele.style.left = '0px'
						if(seqDrag.panel) {
							seqDrag.moveImg.style.left = '0px'
						}
						seqDrag.indicator.style.width = '0px'
					} else if(nowX >= seqDrag.maxLeft) {
						seqDrag.ele.style.left = seqDrag.maxLeft + 'px'
						if(seqDrag.panel) {
							if(nowX >= seqDrag.imgMaxLeft) {
								seqDrag.moveImg.style.left = seqDrag.imgMaxLeft + 'px'
							}
						}
						seqDrag.indicator.style.width = seqDrag.maxLeft + 20 + 'px'
					}	else {
						seqDrag.ele.style.left=nowX+'px';
						if(seqDrag.panel) {
							if(nowX >= seqDrag.imgMaxLeft) {
								seqDrag.moveImg.style.left = seqDrag.imgMaxLeft + 'px'
							}else {
								seqDrag.moveImg.style.left = nowX + 'px'
							}
						}
						seqDrag.indicator.style.width = nowX + 20 + 'px'
					}
					seqDrag.moveCallBack && seqDrag.moveCallBack()
				}
				break;
			case 'mouseup':
				if(seqDrag.toggle){
					let success= false
					seqDrag.toggle=false;
					if(seqDrag.panel) {  //拼图
						if(parseInt(seqDrag.ele.style.left) >= seqDrag.range[0] && parseInt(seqDrag.ele.style.left) <= seqDrag.range[1]) {
							console.log('成功')
							success = true
							seqDrag.upCallBack && seqDrag.upCallBack(success)
						}else if(parseInt(seqDrag.ele.style.left) != 0) {
							seqDrag.fail = true
							setTimeout(() => {
								seqDrag.ele.style.left = '0px'
								seqDrag.moveImg.style.left = '0px'
								seqDrag.fail = false
							}, 400)	
							console.log('失败')
							success = false
							seqDrag.upCallBack && seqDrag.upCallBack(success)
						}
					}else {
						if(parseInt(seqDrag.ele.style.left) === seqDrag.maxLeft) {
							console.log('成功')
							success = true
							seqDrag.upCallBack && seqDrag.upCallBack(success)
						}else if(parseInt(seqDrag.ele.style.left) != 0) {
							seqDrag.fail = true
							setTimeout(() => {
								seqDrag.ele.style.left = '0px'
								seqDrag.fail = false
							}, 400)	
							console.log('失败')
							success = false
							seqDrag.upCallBack && seqDrag.upCallBack(success)
						}
					}
					
				}
				break;
			default:break;
		}
	}
	static panelHandler(ev) {
		if(ev.type == 'mouseenter') {
			seqDrag.panelCallBack && seqDrag.panelCallBack(true)
			seqDrag.panel.style.display = 'block'
		}else if(ev.type == 'mouseleave') {
			if(!seqDrag.toggle && !seqDrag.fail) {
				seqDrag.panelCallBack && seqDrag.panelCallBack(false)
				setTimeout(() => {
					seqDrag.panel.style.display = 'none'
				}, 200)	
			}
		}
	}
}
const random = function() {
	return Math.floor((Math.random()*5));
}

	export default {
		props: {
			puzzle: {
				type: Boolean,
				default: true
			}
		},
		data() {
			return {
				indicatorWidth: 0,
				sliderLeft: 0,
				isShowTip: true,
				isMove: false,
				isFail: false,
				isSuccess: false,

				isPanelActive: '0',

				imgList: [{
					bgImg: 'https://necaptcha.nosdn.127.net/3571b0f532384d0b837873e1419a0d22.jpg',
					moveImg: 'https://necaptcha.nosdn.127.net/ebf9a57ef0194c99841c488575fc76b4.png',
					range: [175, 185]
				}, {
					bgImg: 'https://necaptcha.nosdn.127.net/93c76147fa304ec8babaec3c4bd175dd.jpg',
					moveImg: 'https://necaptcha.nosdn.127.net/900267569a764fe7a44f837ba8aaa0d2.png',
					range: [132, 142]
				}, {
					bgImg: 'https://necaptcha.nosdn.127.net/b2f501a5845f4bcf8e7e2d4752125141.jpg',
					moveImg: 'https://necaptcha.nosdn.127.net/b4ddefb998cd464886e6171dcdbbec3c.png',
					range: [108, 118]
				}, {
					bgImg: 'https://necaptcha.nosdn.127.net/3d6a0abceabe478d8a9c6b49979d5507.jpg',
					moveImg: 'https://necaptcha.nosdn.127.net/b4df1f72797c406b8f2424d3413ce468.png',
					range: [183, 193]
				}, {
					bgImg: 'https://necaptcha.nosdn.127.net/e3b2f720ac9048a888525dc52a83cc20.jpg',
					moveImg: 'https://necaptcha.nosdn.127.net/13606f26398442b18a2360ab1d15e509.png',
					range: [250, 260]
				}],
				imgData: null
			}
		},
		created() {
			if(this.puzzle) {
				this.imgData = this.imgList[random()]
			}
		},
		mounted() {
			this.init()
		},
		methods: {
			init() {
				seqDrag.init({
					move: () => {
						this.isShowTip = false
						this.isMove = true
					},
					up: (flag) => {
						this.isShowTip = true
						this.isMove = false
						if(flag) {
							this.isSuccess = true
							this.isPanelActive = '2'
							seqDrag.close()
						}else {
							this.isFail = true
							setTimeout(() => {
								this.isFail = false
								if(this.puzzle) {
									this.imgData = this.imgList[random()]
									seqDrag.reset(this.imgData.range)
								}
							}, 400)
						}
					},
					panel: (flag) => {
						if(flag) {
							this.isPanelActive = '1'
						}else {
							this.isPanelActive = '2'
						}
					}
				}, this.puzzle ? this.imgData.range : [])
			}
		}
	}
</script>
<style lang="less" scoped>
.animated {
  -webkit-animation-duration: .2s;
  animation-duration: .2s;
  -webkit-animation-fill-mode: both;
  animation-fill-mode: both;
}
@-webkit-keyframes fadeInUp {
  from {
    opacity: 0;
    -webkit-transform: translate3d(0, 15px, 0);
    transform: translate3d(0, 15px, 0);
  }

  to {
    opacity: 1;
    -webkit-transform: translate3d(0, 0, 0);
    transform: translate3d(0, 0, 0);
  }
}
@keyframes fadeInUp {
  from {
    opacity: 0;
    -webkit-transform: translate3d(0, 15px, 0);
    transform: translate3d(0, 15px, 0);
  }

  to {
    opacity: 1;
    -webkit-transform: translate3d(0, 0, 0);
    transform: translate3d(0, 0, 0);
  }
}
@-webkit-keyframes fadeInDown {
  from {
    opacity: 1;
    -webkit-transform: translate3d(0, 0, 0);
    transform: translate3d(0, 0, 0);
  }

  to {
    opacity: 0;
    -webkit-transform: translate3d(0, 15px, 0);
    transform: translate3d(0, 15px, 0);
  }
}

@keyframes fadeInDown {
  from {
    opacity: 1;
    -webkit-transform: translate3d(0, 0, 0);
    transform: translate3d(0, 0, 0);
  }

  to {
    opacity: 0;
    -webkit-transform: translate3d(0, 15px, 0);
    transform: translate3d(0, 15px, 0);
  }
}
.fadeInUp {
  -webkit-animation-name: fadeInUp;
  animation-name: fadeInUp;
}
.fadeInDown {
  -webkit-animation-name: fadeInDown;
  animation-name: fadeInDown;
}
	.dragValite {
		width: 320px;
		text-align: center;
		.dragValite-panel {
			display: none;
	    position: absolute;
	    left: 0;
	    // width: 100%;
	    z-index: 999;
	    user-select: none;
			width: 320px;
			height: 160px;
			margin-bottom: 15px;
			bottom: 40px;
			.dragValite-placeholder {
				pointer-events: auto;
		    position: relative;
		    padding-top: 50%;
		    .dragValite_bgimg {
		    	border-radius: 2px;
		    	pointer-events: auto;
			    position: absolute;
			    top: 0;
			    left: 0;
			    width: 100%;
			    height: 100%;
			    .dragValite_bg-img {
			    	border-radius: 2px;
			    	pointer-events: auto;
				    vertical-align: top;
				    width: 100%;
			    }
			    .dragValite_lyjmoveimg {
			    	display: block;
			    	position: absolute;
				    left: 0;
				    top: 0;
				    width: auto;
				    height: 100%;
			    }
		    }
			}
		}
		.dragValite-control {
			height: 40px;
			border: 1px solid #e4e7eb;
			border-radius: 2px;
			background-color: #f7f9fa;
			position: relative;
			.dragValite_slide_indicator {
				box-sizing: border-box;
				height: 40px;
    		border-radius: 2px;
    		width: 0px;
    		position: absolute;
    		// top: -1px;
    		left: -1px;
    		border: 1px solid transparent;
    		&.__move {
    			border-color: #199afa;
    			background-color: #d1e9fe;
    		}
    		&.__success {
    			border-color: #52ccba;
    			background-color: #d2f4ef;
    		}
    		&.__fail {
    			border-color: #f57a7a;
    			background-color: #fce1e1;
    		}
			}
			.dragValite_slider {
				width: 40px;
				border-radius: 2px;
				position: absolute;
		    top: 0;
		    left: 0;
		    height: 100%;
		    background-color: #fff;
		    box-shadow: 0 0 3px rgba(0,0,0,.3);
		    cursor: pointer;
		    transition: background .2s linear;
		    &:hover {
		    	background-color: #199afa;
		    	.dragValite_slider__icon {
			    	background-image: url(https://cstaticdun.126.net//2.14.0/images/icon_light.b6bfcff.png);
				    background-position: 0 0;
				    background-size: 32px 576px;
				  }
		    }
		    &.__success {
		    	background-color: #52ccba;
		    	.dragValite_slider__icon {
		    		background-position: 0 -26px;
		    	}
		    	&:hover {
		    		background-color: #52ccba;
		    		.dragValite_slider__icon {
			    		background-position: 0 -26px;
			    	}
		    	}
		    }
		    &.__fail {
		    	background-color: #f57a7a;
		    	.dragValite_slider__icon {
		    		background-position: 0 -83px;
		    	}
		    	&:hover {
		    		background-color: #f57a7a;
		    		.dragValite_slider__icon {
			    		background-position: 0 -83px;
			    	}
		    	}
		    }
		    .dragValite_slider__icon {
	        position: absolute;
			    top: 50%;
			    margin-top: -6px;
			    left: 50%;
			    margin-left: -6px;
			    width: 14px;
			    height: 10px;
			    background-image: url(https://cstaticdun.126.net//2.14.0/images/icon_light.b6bfcff.png);
			    background-position: 0 -13px;
			    background-size: 32px 576px;
				}
			}
			.dragValite_tip {
				line-height: 40px;
				text-align: center;
    		color: #45494c;
    		.dragValite_tip-text {
					vertical-align: middle;
					user-select: none;
    		}
			}
		}
	}
</style>