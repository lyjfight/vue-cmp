<template>
	<div class="dragValite">
		<div class="dragValite-control">
			<div class="dragValite_slide_indicator" :class="{__move: isMove, __success: isSuccess}" :style="{width: indicatorWidth + 'px'}"></div>
			<div class="dragValite_slider" :class="{__success: isSuccess}" :style="{left: sliderLeft + 'px'}">
				<span class="dragValite_slider__icon"></span>
			</div>
			<div class="dragValite_tip">
				<span class="dragValite_tip-text" v-show="isShowTip">
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
	static init(downCallBack, moveCallBack, upCallBack) {
		seqDrag.slider=document.querySelector('.dragValite_slider');
		seqDrag.control=document.querySelector(".dragValite-control");
		seqDrag.indicator=document.querySelector(".dragValite_slide_indicator");

		seqDrag.slider.style.left = seqDrag.slider.offsetLeft + 'px'

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
	}
	static close() {
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
					if(nowX <= 0) {
						seqDrag.ele.style.left = '0px'
						seqDrag.indicator.style.width = '0px'
					} else if(nowX >= seqDrag.maxLeft) {
						seqDrag.ele.style.left = seqDrag.maxLeft + 'px'
						seqDrag.indicator.style.width = seqDrag.maxLeft + 20 + 'px'
					}	else {
						seqDrag.ele.style.left=nowX+'px';
						seqDrag.indicator.style.width = nowX + 20 + 'px'
					}
					seqDrag.moveCallBack && seqDrag.moveCallBack()
				}
				break;
			case 'mouseup':
				if(seqDrag.toggle){
					let success= false
					seqDrag.toggle=false;
					if(parseInt(seqDrag.ele.style.left) === seqDrag.maxLeft) {  //滑到底成功
						console.log('成功')
						success = true
					}else {
						seqDrag.ele.style.left = '0px'
						console.log('失败')
						success = false
					}
					seqDrag.upCallBack && seqDrag.upCallBack(success)
				}
				break;
			default:break;
		}
	}
}

	export default {
		data() {
			return {
				indicatorWidth: 0,
				sliderLeft: 0,
				isShowTip: true,
				isMove: false,
				isSuccess: false
			}
		},
		mounted() {
			seqDrag.init(() => {

			}, () => {
				this.isShowTip = false
				this.isMove = true
			}, (flag) => {
				this.isShowTip = true
				this.isMove = false
				if(flag) {
					this.isSuccess = true
					seqDrag.close()
				}
			})
		}
	}
</script>
<style lang="less" scoped>
	.dragValite {
		width: 280px;
		text-align: center;
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
    		top: -1px;
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
		    	background-color: #d2f4ef;
		    	&:hover {
		    		background-color: #d2f4ef;
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