<template>
	<view class="imageUploadContainer">
		<view class="imageUploadList">
			<view class="imageItem" v-bind:key="index" v-for="(path,index) in imageListData">
				<image :src="path" :class="{'dragging':isDragging(index)}" draggable="true" @tap="previewImage" :data-index="index" @touchstart="start" @touchmove.stop.prevent="move" @touchend="stop"></image>
				<view v-if="isShowDel" class="imageDel" @tap="deleteImage" :data-index="index"> </view>
			</view>
			<view v-if="isShowAdd" class="imageUpload" @tap="selectImage">+</view>
		</view>
		<image v-if="showMoveImage" class="moveImage" :style="{left:posMoveImageLeft, top:posMoveImageTop}" :src="moveImagePath"></image>
	</view>
</template>

<script>
	export default {
		name:'robby-image-upload',
		props: ['value','enableDel','enableAdd','enableDrag','serverUrl','formData','header', 'limit','fileKeyName','showUploadProgress','serverUrlDeleteImage'],
		data() {
			return {
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
				targetImageIndex: null,
				imageList: [],
				isDestroyed: false
			}
		}, 
		mounted: function(){
			this.imageList = this.value
			
			if(this.showUploadProgress === false){
				this.showUploadProgress = false
			}else{
				this.showUploadProgress = true
			}
		},
		destroyed: function(){
			this.isDestroyed = true
		},
		computed:{
			imageListData: function(){
				if(this.value){
					return this.value
				}
			},
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
				}
				
				if(this.limit && this.imageList.length >= this.limit){
					return false
				}
				
				return true
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
				var _self = this
				if(!_self.imageList){
					_self.imageList = []
				} 
				
				uni.chooseImage({
					count: _self.limit ? (_self.limit - _self.imageList.length) : 999,
					success: function(e){
						var imagePathArr = e.tempFilePaths
						
						//如果设置了limit限制，在web上count参数无效，这里做判断控制选择的数量是否合要求
						//在非微信小程序里，虽然可以选多张，但选择的结果会被截掉
						//在app里，会自动做选择数量的限制
						if(_self.limit){
							var availableImageNumber = _self.limit - _self.imageList.length
							if(availableImageNumber < imagePathArr.length){
								uni.showToast({
									title: '图片总数限制为'+_self.limit+'张，当前还可以选'+availableImageNumber+'张',
									icon:'none',
									mask: false,
									duration: 2000
								});
								return
							}
						}
						
						//检查服务器地址是否设置，设置即表示图片要上传到服务器
						if(_self.serverUrl){
							uni.showToast({
								title: '上传进度：0/' + imagePathArr.length,
								icon: 'none',
								mask: false
							});
							
							var remoteIndexStart = _self.imageList.length - imagePathArr.length
							var promiseWorkList = []
							var keyname = (_self.fileKeyName ? _self.fileKeyName : 'upload-images')
							var completeImages = 0
							
							for(let i=0; i<imagePathArr.length;i++){
								promiseWorkList.push(new Promise((resolve, reject)=>{
									let remoteUrlIndex = remoteIndexStart + i
									uni.uploadFile({
										url:_self.serverUrl,
										fileType: 'image',
										header: _self.header,
										formData:_self.formData,
										filePath: imagePathArr[i], 
										name: keyname,
										success: function(res){
											if(res.statusCode === 200){
												if(_self.isDestroyed){
													return
												}
												
												completeImages ++
												
												if(_self.showUploadProgress){
													uni.showToast({
														title: '上传进度：' + completeImages + '/' + imagePathArr.length,
														icon: 'none',
														mask: false,
														duration: 500
													});
												}
												console.log('success to upload image: ' + res.data)
												resolve(res.data)
											}else{
												console.log('fail to upload image:'+res.data)
												reject('fail to upload image:' + remoteUrlIndex)
											}
										},
										fail: function(res){
											console.log('fail to upload image:'+res)
											reject('fail to upload image:' + remoteUrlIndex)
										}
									})
								}))
							}
							Promise.all(promiseWorkList).then((result)=>{
								if(_self.isDestroyed){
									return
								}
								
								for(let i=0; i<result.length;i++){
									_self.imageList.push(result[i])
								}
								
								_self.$emit('add', {
									currentImages: imagePathArr,
									allImages: _self.imageList
								})
								_self.$emit('input', _self.imageList)
							})
						}else{
							for(let i=0; i<imagePathArr.length;i++){
								_self.imageList.push(imagePathArr[i])
							}
							
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
				
				//检查删除图片的服务器地址是否设置，如果设置则调用API，在服务器端删除该图片
				if(this.serverUrlDeleteImage){
					uni.request({
						url: this.serverUrlDeleteImage,
						method: 'GET',
						data: {
							imagePath: deletedImagePath
						},
						success: res => {
							console.log(res.data)
						}
					});
				}
				
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
				var _self = this
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
				console.log(this.isDragable)
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
				
				this.$emit('input', this.imageList)
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

	.imageItem{
		position: relative;
	}
	
	.imageDel{
		position: absolute;
		top: -5px;
		right: -5px;
		width: 20px;
		height: 20px;
		border-radius: 30px;
		box-shadow: 0 0 5px #999;
		background: url("data:image/png;base64,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")
					center no-repeat ;
		background-size: cover;
	}
	
	.imageItem image, .moveImage{
		width: 160upx;
		height: 160upx;
		border-radius: 8upx;
	}
	
	.imageUpload{
		line-height: 130upx;
		text-align: center;
		font-size: 150upx;
		color: #D9D9D9;
		border: 1px solid #D9D9D9;
		border-radius: 8upx;
	}
	
	.moveImage{
		position: absolute;
	}
</style>
