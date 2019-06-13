<template>
	<view class="">
		<robby-image-upload v-model="imageData" :server-url-delete-image="serverUrlDeleteImage" :server-url="serverUrl" :showUploadProgress="show" :form-data="formData" @delete="deleteImage" @add="addImage"></robby-image-upload>
	</view>
</template>

<script>
	import robbyImageUpload from '@/components/robby-image-upload/robby-image-upload.vue'
	export default {
		data() {
			return {
				enableDel : false,
				enableAdd : false,
				enableDrag : false,
				show: true,
				imageData : [],
				serverUrl: 'http://localhost:2000/work/uploadWorkPicture',
				serverUrlDeleteImage: 'http://localhost:2000/work/deleteWorkPicture',
				formData: {
					userId: 2
				}
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
