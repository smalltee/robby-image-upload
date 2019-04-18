<template>
	<view class="">
		<robby-image-upload v-model="imageData" :server-url="serverUrl" :form-data="formData" @delete="deleteImage" @add="addImage"></robby-image-upload>
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
				imageData : [],
				serverUrl: 'http://localhost:3000/work/uploadWorkPicture',
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
