<template>
  <div class="dialog_win">
	  <div class="dialog_north">
	    <span>图片裁剪</span>
	    <div class="dialog_close" @click="hide"></div>
	  </div>
	  <div class="dialog_ifm_loading" style="width: 594px; height: 0px;"></div>
	  <div class="dialog_ifm" style="height: 415px;">
	  	
			<div class="box">
			  <div id="browse" class="browse">
			    <div id="upload" style="width: 150px; height: 30px; overflow: hidden; opacity: 0.001; position: absolute; background-color: blue;">
			      <input type="file" accept="image/jpeg,image/jpg,image/bmp,image/png" style="cursor: pointer; width: 500px; height: 30px; margin-left: -200px; opacity: 0;"></div>
			    <a>选择图片</a>
			  </div>
			  <div id="raw" class="raw">
			  	
					<div class="photo_cutter_outer" ref="photoOuter">
					  <div class="photo_cutter_mask" ref="photoMask">

					    <div class="photo_cutter_mask_sub photo_cutter_mask_north" :style="{height:maskTopPos+'px'}"></div>
					    <div class="photo_cutter_mask_sub" :style="{top:maskTopPos+'px',width:maskLeftPos+'px',height:maskCenter+'px'}"></div>
					    <div class="photo_cutter_mask_sub photo_cutter_mask_east"  :style="{top:maskTopPos+'px',width:maskRightPos+'px',height:maskCenter+'px'}"></div>
					    <div class="photo_cutter_mask_sub photo_cutter_mask_south" :style="{height:maskBottomPos+'px'}"></div>


					    <div class="photo_cutter_center" ref="maskCenter" :style="{top:maskTopPos+'px',left:maskLeftPos+'px',width:maskCenter+'px',height:maskCenter+'px'}" @mousedown.stop.prevent="bindMaskMouseDown($event)">
					      <div class="photo_cutter_fake"></div>
					      <div class="photo_cutter_block photo_cutter_block_0" @mousedown.stop.prevent="bindDrawMouseDown(1,$event)"></div>
					      <div class="photo_cutter_block photo_cutter_block_2" @mousedown.stop.prevent="bindDrawMouseDown(2,$event)"></div>
					      <div class="photo_cutter_block photo_cutter_block_5" @mousedown.stop.prevent="bindDrawMouseDown(3,$event)"></div>
					      <div class="photo_cutter_block photo_cutter_block_7" @mousedown.stop.prevent="bindDrawMouseDown(4,$event)"></div>
					    </div>
					  </div>
					  <div class="photo_cutter_tip" style="line-height: 256px; display: none;">loading...</div>
					  <img class="orgImg" ref="orgImg" :src="orgImgSrc" @load="getOrgImgOpt">
					</div>

			  </div>
			  <div id="preview" class="preview">
					<img class="newImg" ref="newImg" :src="orgImgSrc">
			  </div>
			  
			</div>

	  </div>
	  
	</div>
</template>

<script>
export default {
  data(){
  	return {
  		wrapWidth : 320,//外框
  		wrapHeight : 320,//外框
  		newImgwrapWidth : 200,//新图预览外框
  		newImgwrapHeight : 200,
  		orgImgSrc:'',//加载的原图
  		orgImgWidth : 0,//原图宽高
  		orgImgHeight : 0,
  		curWidth : 0,//设置外框后的宽高
  		curHeight : 0,//设置外框后的宽高
  		maskCenter : 150,//遮罩内框宽高
  		defMaskCenter : 150,//记录遮罩内框宽高,用于拖拽
  		maskTopPos : 0,//遮罩
  		maskLeftPos : 0,
  		maskRightPos : 0,
  		maskBottomPos : 0,
  		mousemoveLock : false,//按下时才触发
  		drawLock : false,//推拽
  		downX : 0,
  		downY : 0,
  		curX : 0,
  		curY : 0,
  		drawX : 0,
  		drawY : 0,
  		curDrwaX : 0,//拖拽用改变位置
  		curDrwaY : 0,
  		drawType : 0//拖拽位置
  	}
  },
  mounted(){
  	this.loadImg('../static/images/2.jpg')
  },
  methods:{
  	hide(){

    },
  	loadImg(URL){
  		this.orgImgSrc = URL
  	},
  	getOrgImgOpt(){
  		//获取图片信息
  		let img = this.$refs.orgImg
  		this.orgImgWidth = img.offsetWidth
  		this.orgImgHeight = img.offsetHeight



  		if(this.orgImgWidth>this.orgImgHeight){
  			img.width = this.wrapWidth
  			img.height = this.wrapWidth*this.orgImgHeight/this.orgImgWidth
  			this.curWidth = this.wrapWidth
  			this.curHeight = img.height
  		}else{
  			img.width = this.orgImgWidth*this.wrapHeight/this.orgImgHeight
  			img.height = this.wrapHeight
  			this.curWidth = img.width
  			this.curHeight = this.wrapHeight
  		}
  		this.setMaskStyle()

  	},
  	setMaskStyle(){
  		//设置遮罩样式

  		this.$refs.photoOuter.style.width = this.curWidth+'px'
  		this.$refs.photoOuter.style.marginTop = (this.wrapHeight-this.curHeight)/2+'px'

  		this.$refs.photoMask.style.width = this.curWidth+'px'
  		this.$refs.photoMask.style.height = this.curHeight+'px'

  		this.setNewImgStyle()
  		this.setMaskPos()
  	},
  	setNewImgStyle(){
  		//设置新图预览
  		let proX = this.maskCenter/this.curWidth//比例
  		let proY = this.maskCenter/this.curHeight
  		let newImg = this.$refs.newImg
  		newImg.style.width = this.newImgwrapWidth/proX+'px'
  		newImg.style.height = this.newImgwrapHeight/proY+'px'


  		this.setNewImgPos()

  	},
  	setNewImgPos(){
  		let newImg = this.$refs.newImg
  		let proX = this.curX/this.maskCenter
  		let proY = this.curY/this.maskCenter
  		newImg.style.left = -this.newImgwrapWidth*proX+'px'
  		newImg.style.top = -this.newImgwrapWidth*proY+'px'
  	},
  	setMaskPos(){
  		//设置遮罩位置
  		this.maskTopPos = this.curY
  		this.maskLeftPos = this.curX
  		this.maskRightPos = this.curWidth-this.maskCenter-this.maskLeftPos
  		this.maskBottomPos = this.curHeight-this.maskCenter-this.maskTopPos
  	},
  	bindMaskMouseDown(event){
  		let mc = this.$refs.maskCenter
  		this.downX = event.clientX - mc.offsetLeft
  		this.downY = event.clientY - mc.offsetTop
			this.mousemoveLock = true

      document.addEventListener('mousemove', this.bindMaskMouseMove)
      document.addEventListener('mouseup', this.bindMaskMouseUp)

  	},
  	bindMaskMouseMove(event){
  		if(!this.mousemoveLock){
  			return
  		}

  		let x = event.clientX-this.downX
  		let y = event.clientY-this.downY
  		let maxX = this.curWidth-this.maskCenter
  		let maxY = this.curHeight-this.maskCenter
  		if(x<0){
  			x = 0
  		}
  		if(y<=0){
  			y = 0
  		}
  		if(x>maxX){
  			x = maxX
  		}
  		if(y>maxY){
  			y = maxY
  		}
  		this.curX = this.curDrwaX = x
  		this.curY = this.curDrwaY = y
  		this.setMaskPos()

  		this.setNewImgPos()

  	},
  	bindMaskMouseUp(){
  		this.mousemoveLock = false
      document.removeEventListener('mousemove',this.bindMaskMouseMove,false);
      document.removeEventListener('mouseup',this.bindMaskMouseUp,false);
  	},
  	bindDrawMouseDown(type,event){
  		// 缩放
  		this.drawLock = true
  		this.drawType = type
  		this.drawX = event.clientX
  		this.drawY = event.clientY

  		document.addEventListener('mousemove', this.bindDrawMouseMove)
      document.addEventListener('mouseup', this.bindDrawMouseUp)

  	},
  	bindDrawMouseMove(event){
  		if(!this.drawLock){
  			return
  		}
  		let addW = event.clientX - this.drawX

  		if(this.drawType===1){
  			//左上

  			if((this.defMaskCenter-addW)+this.maskRightPos>=this.curWidth){
	  			addW = this.defMaskCenter - this.curWidth + this.maskRightPos
	  		}
  			if((this.defMaskCenter-addW)+this.maskBottomPos>=this.curHeight){
	  			addW = this.defMaskCenter - this.curHeight + this.maskBottomPos
	  		}
  			
	  		this.maskCenter = -addW+this.defMaskCenter
	  		this.curX = this.curDrwaX+addW
	  		this.curY = this.curDrwaY+addW
  		}else if(this.drawType===2){
  			//右上

  			if(this.defMaskCenter+addW+this.maskLeftPos>=this.curWidth){
	  			addW = this.curWidth - this.defMaskCenter - this.maskLeftPos
	  		}
  			if(this.defMaskCenter+addW+this.maskBottomPos>=this.curHeight){
	  			addW = this.curHeight - this.defMaskCenter - this.maskBottomPos
	  		}
  			
	  		this.maskCenter = addW+this.defMaskCenter
	  		this.curY = this.curDrwaY-addW
  		}else if(this.drawType===3){
  			//左下

  			if((this.defMaskCenter-addW)+this.maskRightPos>=this.curWidth){
	  			addW = this.defMaskCenter - this.curWidth + this.maskRightPos
	  		}
  			if((this.defMaskCenter-addW)+this.maskTopPos>=this.curHeight){
	  			addW = this.defMaskCenter - this.curHeight + this.maskTopPos
	  		}
  			
	  		this.maskCenter = -addW+this.defMaskCenter
	  		this.curX = this.curDrwaX+addW
  		}else if(this.drawType===4){
  			//右下
  			if(addW+this.defMaskCenter+this.maskLeftPos>=this.curWidth){
	  			addW = this.curWidth - this.defMaskCenter - this.maskLeftPos
	  		}
	  		if(addW+this.defMaskCenter+this.maskTopPos>=this.curHeight){
	  			addW = this.curHeight - this.defMaskCenter - this.maskTopPos
	  		}

	  		this.maskCenter = addW+this.defMaskCenter

  		}
  		
  		this.setMaskPos()
	  	this.setNewImgStyle()

  	},
  	bindDrawMouseUp(){
  		this.defMaskCenter = this.maskCenter
  		this.curDrwaX = this.curX
  		this.curDrwaY = this.curY
  		this.drawLock = false
  		document.removeEventListener('mousemove',this.bindDrawMouseMove,false);
      document.removeEventListener('mouseup',this.bindDrawMouseUp,false);
  	}

  }
}

</script>

<style scoped>
.dialog_win{
	width:600px;
	height:500px;
	border-radius:5px;
	position:fixed;
	left:50%;
	top:50%;
	z-index:100;
	margin:-250px 0 0 -300px;
  box-shadow: 0 0 20px black;
}
.box{
	padding:20px;
	overflow: hidden;
	background:#F5F5F5;
}

.dialog_north {
  color: white;
  height: 45px;
  line-height: 45px;
  text-align: left;
  text-indent: 10px;
  border-bottom: 1px solid #4D6FB4;
  border-radius: 5px 5px 0 0;
  background:#6187D3;
}

.dialog_north span {
  float: left;
  font-size: 15px;
  font-weight: bolder;
}

.dialog_close {
  float: right;
  width: 40px;
  height: 30px;
  margin-top: 10px;
  cursor: pointer;
  background: url("../assets/close.png") no-repeat;
}

.dialog_close:hover {
  background-position: 0 -30px;
}

.dialog_ifm {
  width: 100%;
  margin: 0;
  padding: 0;
  border: 0;
  display: block;
  border-radius: 3px;
  background-color: white;
}

.dialog_ifm_loading {
  position: fixed;
  background: url("../assets/loading2.gif") white no-repeat center 40%;
}

.dialog_south {
  height: 45px;
  text-align: right;
  overflow: hidden;
  background-color: #e6e9ee;
  border-top: 1px solid #dbdbdb;
  border-radius: 0 0 5px 5px;
  margin-top: -4px;
}

.dialog_btn {
  height: 30px;
  cursor: pointer;
  margin-top: 8px;
  margin-right: 20px;
  text-align: center;
  padding: 0 25px;
  border-radius: 3px;
  outline-style: none;
}

.dialog_btn_gray {
  color: #545454;
  border: 1px solid #B3B3B3;
  background-image: linear-gradient(bottom, #E4E4E4, #FAFAFA);
}

.dialog_btn_gray:hover {
  background-image: linear-gradient(bottom, #FAFAFA, #E4E4E4); 
}

.dialog_btn_blue {
  color: white;
  border: 1px solid #4D6FB4;
  background-image: linear-gradient(bottom, #5073b0, #7a9de2); 
}

.dialog_btn_blue:hover {
  background-image: linear-gradient(bottom, #7a9de2, #5073b0); 
}

.dialog_ifm_words {
  float: left;
  padding: 4px 3px 3px 10px
}

.dialog_ifm_words input {
  height: 20px;
  line-height: 20px;
}

input::-moz-focus-inner {
  border: 0;
}

.dialog_win_photo {
  padding: 10px;
  background: white;
  overflow: hidden;
}

.dialog_win_photo img {
  border-bottom: 1px solid #E0E0E0;
  vertical-align: middle;
}
.file {
  position: absolute;
  font-size: 30px;
  width: 100px;
  height: 28px;
  filter: alpha(opacity=0);
  opacity: 0.0;
  cursor: pointer;
  outline: none;
}

.flash {
  filter: alpha(opacity=1);
  opacity: 0.01;
  position: absolute;
}

.browse {
  margin-bottom: 20px;
  width: 150px;
  height: 30px;
  line-height: 30px;
  text-align: center;
  cursor: pointer;
  border-radius: 3px;
  border: 1px solid #B3B3B3;
  background-image: linear-gradient(bottom, #E4E4E4, #FAFAFA); 
}

.browse:hover {
  background-image: linear-gradient(bottom, #FAFAFA, #E4E4E4); 
}

.raw {
  width: 320px;
  height: 320px;
  margin-right: 30px;
  float: left;
  border: 1px solid #D3D3D3;
  background-image: url("../assets/headbackground.png");
}

.preview {
  width: 200px;
  height: 200px;
  border: 1px solid #D3D3D3;
  float: left;
  background: url("../assets/gifthead.png") no-repeat center white;
}


.photo_cutter_outer {
  margin: 0 auto;
  padding: 0;
  font-size: 0;
  overflow: hidden;
}

.photo_cutter_tip {
  font-size: 15px;
  text-align: center;
  margin: 0 auto;
  font-family: cursive;
  font-weight: bold;
}

.photo_cutter_mask {
  position: absolute;
  -moz-user-select:none;
}

.photo_cutter_mask_sub {
  background:black;
  filter: alpha(opacity=70);
  opacity: 0.7;
  position: absolute;
  font-size: 0;
}

.photo_cutter_mask_north {
  width: 100%;
}

.photo_cutter_mask_south {
  width: 100%;
  bottom: 0;
}

.photo_cutter_mask_east {
  right: 0;
}

.photo_cutter_photo {
  filter: alpha(opacity=0);
  opacity: 0;
}

/*.photo_cutter_preview_outer{
  overflow: hidden;
}*/

.photo_cutter_center {
  position: absolute;
  font-size: 0;
  /*border: 1px dotted #0066FF;*/
  padding: 0;
  margin: 0;
}
.photo_cutter_fake {
  cursor: move;
  background-color: #000000;
  filter: alpha(opacity=0);
  opacity: 0;
  height: 100%;
}
.photo_cutter_block {
  height: 6px;
  width: 6px;
  position: absolute;
  border:1px solid #0066FF;
  background:#12CBFA;
}

.photo_cutter_block_0 {
  left: 0;
  top: 0;
  cursor: se-resize;
  border-top: none;
  border-left: none;

}

.photo_cutter_block_1 {
  top: 0;
  cursor: s-resize;
  border-top: none;
}

.photo_cutter_block_2 {
  cursor: ne-resize;
  right: 0;
  top: 0;
  border-top: none;
  border-right: none;
}

.photo_cutter_block_3 {
  cursor: w-resize;
  border-left: none;
}

.photo_cutter_block_4 {
  cursor: e-resize;
  right: 0;
  border-right: none;
}

.photo_cutter_block_5 {
  cursor: ne-resize;
  left: 0;
  bottom: 0;
  border-bottom: none;
  border-left: none;
}

.photo_cutter_block_6 {
  cursor: s-resize;
  bottom: 0;
  border-bottom: none;
}

.photo_cutter_block_7 {
  cursor: se-resize;
  right: 0;
  bottom: 0;
  border-bottom: none;
  border-right: none;
}
.preview{
	overflow: hidden;
	position:relative;
}
.newImg{
	position:absolute;
}


</style>
