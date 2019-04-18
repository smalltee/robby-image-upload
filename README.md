# 项目说明
本项目是基于uni-app的一个通用组件。可支持小程序，也支持APP。组件名称为“图片选择组件”，可添加，可删除，可预览，可拖动调整顺序

上传到服务器功能暂未实现，后续会增加。目前因为选择的图片本地路径已经有了，如果急需要支持，也可以自己先处理。

# 组件功能说明：
1. 可选择多张图片
2. 选择图片后自动添加到后面
4. 图片上自带删除图标，点击可删除
5. 删除图标可配置显示/隐藏
7. 图片可点击预览
8. 图片可拖动重新排序
9. 图片删除时可触发删除事件
10. 添加图片可配置显示/隐藏
11. 可获取当前的图片地址数组

# 使用说明
拷贝该组件到components目录下之后

在 `script`中引用组件
```
import robbyImageUpload from '@/components/robby-image-upload/robby-image-upload.vue'
export default {
    components: {robbyImageUpload}
}
```

在 `template` 中使用组件：
```
<robby-image-upload :value="imageData"></robby-image-upload>
<robby-image-upload v-model="imageData" @delete="deleteImage" @add="addImage" :enable-drag="enableDrag" :enable-del="enableDel" :enable-add="enableAdd"></robby-image-upload>
```

# 双向绑定说明
v-model: 类型为字符串数组。 即图片地址数组，可用于初始化，当添加/删除操作时，绑定的数据都会同步更新

# 属性说明
属性名|类型|默认值|说明
:-:|:-:|:-:|-
enable-del|Boolean|true|删除图标是否可见，即是否可删除
enable-add|Boolean|true|添加图片操作是否可见，即是否可添加图片
enable-drag|Boolean|true|图片是否可拖动，重新排序
value|Array&lt;String&gt;|[]|初始化的图片数据，可用于单向数据初始化，需要双向绑定可直接用v-model

# 事件说明
事件名称|说明|返回参数
:-:|-|-
add|点击”添加“按钮后触发的事件,返回参数为当前操作添加的图片地址数组和当前所有的图片地址数组|{currentImages:Array&lt;String&gt;,allImages:Array&lt;String&gt;}
delete|点击“x”删除图标后触发的事件,返回参数为当前删除的图片地址和当前所有的图片地址数组|{currentImage:String,allImages:Array&lt;String&gt;}

# demo示例
```
<template>
	<view class="">
		<robby-image-upload v-model="imageData" @delete="deleteImage" @add="addImage"></robby-image-upload>
		<view v-for="(item,index) in imageData" :key="index" class="">
			{{index}}. {{item.substr(-14)}}
		</view>
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
```