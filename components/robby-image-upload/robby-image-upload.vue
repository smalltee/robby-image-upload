<template>
	<view class="imageUploadContainer">
		<view class="imageUploadList">
			<view class="imageItem" v-bind:key="index" v-for="(path,index) in imageList">
				<image :src="path" :class="{'dragging':isDragging(index)}" draggable="true" @tap="previewImage" :data-index="index" @touchstart="start" @touchmove.stop="move" @touchend="stop"></image>
				<view v-if="isShowDel" class="imageDel" @tap="deleteImage" :data-index="index">x</view>
				
			</view>
			
			<view v-if="isShowAdd" class="imageUpload" @tap="selectImage">
				+
			</view>
		</view>
		<image v-if="showMoveImage" class="moveImage" :style="{left:posMoveImageLeft, top:posMoveImageTop}" :src="moveImagePath"></image>
	</view>
</template>

<script>
	var _self;
	
	export default {
		name:'robby-image-upload',
		props: ['value','enableDel','enableAdd','enableDrag','serverUrl','formData'],
		data() {
			return {
				imageList:this.value || [],
				imageBasePos:{
					x0: -1,
					y0: -1,
					w:-1,
					h:-1,
				},
				showMoveImage: false,
				moveImagePath: '',
				moveLeft: 0,
				moveTop: 0,
				deltaLeft: 0,
				deltaTop: 0,
				dragIndex: null,
				targetImageIndex: null
			}
		},
		computed:{
			posMoveImageLeft: function(){ 
				return this.moveLeft + 'px'
			},
			posMoveImageTop: function(){
				return this.moveTop + 'px'
			},
			isShowDel: function(){
				if(this.enableDel === false){
					return false
				}else{
					return true
				}
			},
			isShowAdd: function(){
				if(this.enableAdd === false){
					return false
				}else{
					return true
				}
			},
			isDragable: function(){
				if(this.enableDrag === false){
					return false
				}else{
					return true
				}
			}
		},
		methods:{
			selectImage: function(){
				_self = this
				uni.chooseImage({
					success: function(e){
						var imagePathArr = e.tempFilePaths
						
						for(let i=0; i<imagePathArr.length;i++){
							_self.imageList.push(imagePathArr[i])
						}
						
						//检查服务器地址是否设置，设置即表示图片要上传到服务器
						if(_self.serverUrl){
							
							var remoteIndexStart = _self.imageList.length - imagePathArr.length
							var promiseWorkList = []
							
							for(let i=0; i<imagePathArr.length;i++){
								promiseWorkList.push(new Promise((resolve, reject)=>{
									let remoteUrlIndex = remoteIndexStart + i
									uni.uploadFile({
										url:_self.serverUrl,
										fileType: 'image',
										formData:_self.formData,
										filePath: imagePathArr[i], 
										name: 'upload-images',
										success: function(res){
											if(res.statusCode === 200){
												_self.imageList[remoteUrlIndex] = res.data 
												console.log('success to upload image: ' + res.data)
												resolve('success to upload image:' + remoteUrlIndex)
											}else{
												console.log('fail to upload image:'+res.data)
												reject('failt to upload image:' + remoteUrlIndex)
											}
										},
										fail: function(res){
											console.log('fail to upload image:'+res)
											reject('failt to upload image:' + remoteUrlIndex)
										}
									})
								}))
							}
							Promise.all(promiseWorkList).then((result)=>{
								_self.$emit('add', {
									currentImages: imagePathArr,
									allImages: _self.imageList
								})
								_self.$emit('input', _self.imageList)
							})
						}else{
							_self.$emit('add', {
								currentImages: imagePathArr,
								allImages: _self.imageList
							})
							_self.$emit('input', _self.imageList)
						}
					}
				})
			},
			deleteImage: function(e){
				var imageIndex = e.currentTarget.dataset.index
				var deletedImagePath = this.imageList[imageIndex]
				this.imageList.splice(imageIndex, 1) 
				
				this.$emit('delete',{
					currentImage: deletedImagePath,
					allImages: this.imageList
				})
				this.$emit('input', this.imageList)
			},
			previewImage: function(e){
				var imageIndex = e.currentTarget.dataset.index
				uni.previewImage({
					current: this.imageList[imageIndex],
					indicator: "number",
					loop: "true",
					urls:this.imageList
				})
			},
			initImageBasePos: function(){
				let paddingRate = 0.024
				_self = this
				//计算图片基准位置
				uni.getSystemInfo({
					success: function(obj) {
						let screenWidth = obj.screenWidth
						let leftPadding = Math.ceil(paddingRate * screenWidth)
						let imageWidth = Math.ceil((screenWidth - 2*leftPadding)/4)
						
						_self.imageBasePos.x0 = leftPadding
						_self.imageBasePos.w = imageWidth
						_self.imageBasePos.h = imageWidth
					}
				})
			},
			findOverlapImage: function(posX, posY){
				let rows = Math.floor((posX-this.imageBasePos.x0)/this.imageBasePos.w)
				let cols = Math.floor((posY-this.imageBasePos.y0)/this.imageBasePos.h)
				let indx = cols*4 + rows
				return indx
			},
			isDragging: function(indx){
				return this.dragIndex === indx
			},
			start: function(e){
				if(!this.isDragable){
					return
				}
				this.dragIndex = e.currentTarget.dataset.index
				this.moveImagePath = this.imageList[this.dragIndex]
				this.showMoveImage = true
				
				//计算纵向图片基准位置
				if(this.imageBasePos.y0 === -1){
					this.initImageBasePos()
					
					let basePosY = Math.floor(this.dragIndex / 4) * this.imageBasePos.h
					let currentImageOffsetTop = e.currentTarget.offsetTop
					this.imageBasePos.y0 = currentImageOffsetTop - basePosY
				}
				
				//设置选中图片当前左上角的坐标
				this.moveLeft = e.target.offsetLeft
				this.moveTop = e.target.offsetTop
			},
			move: function(e){
				if(!this.isDragable){
					return
				}
				const touch = e.touches[0]
				this.targetImageIndex = this.findOverlapImage(touch.clientX, touch.clientY)
				
				//初始化deltaLeft/deltaTop
				if(this.deltaLeft === 0){
					this.deltaLeft = touch.clientX - this.moveLeft
					this.deltaTop = touch.clientY - this.moveTop 
				}
				
				//设置移动图片位置
				this.moveLeft = touch.clientX - this.deltaLeft
				this.moveTop = touch.clientY - this.deltaTop
			},
			stop: function(e){
				if(!this.isDragable){
					return
				}
				if(this.dragIndex !== null && this.targetImageIndex !== null){
					if(this.targetImageIndex<0){
						this.targetImageIndex = 0
					}
				
					if(this.targetImageIndex>=this.imageList.length){
						this.targetImageIndex = this.imageList.length-1
					}
					//交换图片
					if(this.dragIndex !== this.targetImageIndex){
						this.imageList[this.dragIndex] = this.imageList[this.targetImageIndex]
						this.imageList[this.targetImageIndex] = this.moveImagePath
					}
				}
				
				this.dragIndex = null
				this.targetImageIndex = null
				this.deltaLeft = 0
				this.deltaTop = 0
				this.showMoveImage = false
			}
		}
	}
</script>

<style>
	.imageUploadContainer{
		padding: 10upx 5upx;
		margin: 10upx 5upx;
	}
	
	.dragging{
		transform: scale(1.2)
	}
	
	.imageUploadList{
		display: flex;
		flex-wrap: wrap;
	}
	
	.imageItem, .imageUpload{
		width: 160upx;
		height: 160upx;
		margin: 10upx;
	}
	
	.imageProgress{
		position: relative;
		left: 10upx;
		bottom: 100upx;
		border: 1px solid black;
		font-size: 40upx;
		color: white;
		width: 60upx;
	}
	
	.imageDel{
		position: relative;
		left: 120upx;
		bottom: 54upx;
		background-color: rgba(255,255,255,0.5);
		width: 36upx;
		text-align: center;
		padding:1upx 1upx 5upx 1upx;
		line-height: 31upx;
		border-radius: 60upx;
		font-size: 40upx;
	}
	
	.imageItem image, .moveImage{
		width: 160upx;
		height: 160upx;
		border-radius: 20upx;
	}
	
	.imageUpload{
		line-height: 130upx;
		text-align: center;
		font-size: 150upx;
		border-radius: 20upx;
	}
	
	.moveImage{
		position: absolute;
	}
</style>