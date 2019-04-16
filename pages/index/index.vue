<template>
	<view class="">
		<robby-image-upload :image-data="imageData" @delete="deleteImage" @add="addImage"></robby-image-upload>
	</view>
</template>

<script>
	import robbyImageUpload from '@/components/robby-image-upload/robby-image-upload.vue'
	export default {
		data() {
			return {
				enableDel : false,
				enableAdd : false,
				imageData : ['http://e0.ifengimg.com/11/2019/0416/C84611AFCFFA54F880452B866733AF65E7E65899_size80_w1280_h960.jpeg', 'http://e0.ifengimg.com/09/2019/0416/3F9235CCC4A216818ED26B23CA2F9DD3D2FE7566_size718_w750_h400.jpeg']
			} 
		},
		components: {robbyImageUpload},
		methods:{
			deleteImage: function(e){
				console.log(e)
			},
			addImage: function(e){
				console.log(e)
			}
		}
	}
</script>

<style>
</style>
