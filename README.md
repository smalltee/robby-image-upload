# 项目说明
本项目是基于uni-app的一个通用组件。可支持小程序，也支持APP。组件名称为“图片选择组件”，可添加，可删除，可预览，可拖动调整顺序，可上传到后台服务器（服务器端需要自己写）

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
12. 可上传图片到服务器
13. 可设置上传的图片数量

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
初始化显示图片
<robby-image-upload :value="imageData"></robby-image-upload>

绑定图片数据，监听添加、删除事件，设置是否拖拉，是否可删除，是否可选择添加，图片数量限制
<robby-image-upload v-model="imageData" @delete="deleteImage" @add="addImage" :enable-drag="enableDrag" :enable-del="enableDel" :enable-add="enableAdd" :limit="limitNumber"></robby-image-upload>

支持图片上传服务器：选择图片后，组件会自动上传到指定地址，并更新组件中保存的图片地址为服务器地址
<robby-image-upload :server-url="serverUrl" :form-data="formData"></robby-image-upload>
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
server-url|String|null|图片上传的服务器地址，为空或不填写表示不上传图片。填写后本组件在选择图片后会自动上传服务器，add/delete事件中的allImages参数会更新为由服务器端传回的图片地址。
server-url-delete-image|String|null|删除图片的服务器地址，为空或不填写表示不需要调用后台完成删除操作。填写后本组件在点击删除按钮后会调用该接口，具体的删除操作需要自行实现。下方有一个Node作为后台删除图片的例子
form-data|Object|null|上传图片到服务器时，如果需要自定义数据，可以通过此属性进行传递。
limit|Number|无|限制总共可上传的图片数量，默认无限制
fileKeyName|String|'upload-images'|用于在服务端通过自定义key值获取该文件数据
showUploadProgress|Boolean|true|是否显示选择图片的上传进度（以提示信息的方式）

# 后台上传图片代码编写说明（可参考下面:后台上传图片demo）
1. 在服务器获取的文件key值默认为“upload-images”，可通过参数fileKeyName来自定义名字
2. 因为微信只支持单张上传，所以此组件的实现为连续单张上传，而不是一次上传多张图片
3. 上传成功后，需要图片的新地址传回来，用于前端更新图片地址

# 后台删除图片代码编写说明（可参考下面:后台删除图片demo）
1. 调用该删除接口时，传输参数为：imagePath。该参数表示删除的图片完整地址（如：http://localhost:1234/2/2834o21.jpg)

# 事件说明
事件名称|说明|返回参数
:-:|-|-
add|点击”添加“按钮后触发的事件,返回参数为当前操作添加的图片地址数组和当前所有的图片地址数组|{ currentImages: Array&lt;String&gt;, allImages: Array&lt;String&gt; }
delete|点击“x”删除图标后触发的事件,返回参数为当前删除的图片地址和当前所有的图片地址数组|{ currentImage: String, allImages: Array&lt;String&gt; }

# demo示例
```
<template>
	<view class="">
		<robby-image-upload v-model="imageData" @delete="deleteImage" @add="addImage" :server-url-delete-image="serverUrlDeleteImage" :server-url="serverUrl"></robby-image-upload>
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
				limitNumber: 8,
				imageData : [],
				serverUrl: 'http://localhost:1234/work/uploadWorkPicture',
				serverUrlDeleteImage: 'http://localhost:1234/work/deleteWorkPicture',
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
```

# 后台接收上传图片/删除图片示例代码(Node JS)
```
var express = require('express');
var router = express.Router();
var path = require('path')
var fs = require('fs')
var multiparty = require('connect-multiparty')
var multipartyMiddleware = new multiparty()

router.post('/uploadWorkPicture', multipartyMiddleware, function(req, res){
	//userId为前端传输的formData中自定义的数据
	var userId = req.body.userId
	var uploadedImage = req.files['upload-images']
	
	//复制上传的文件到指定目录work_material/:userid
	var sourceStream = fs.createReadStream(uploadedImage.path)
	var destStream = fs.createWriteStream('./work_material/'+userId+'/'+uploadedImage.name)
	sourceStream.pipe(destStream)
	
	sourceStream.on('end', function(){
		//删除临时文件
		fs.unlinkSync(uploadedImage.path)
		console.log('Success upload image' + uploadedImage.name)
		//返回给前端的数据，前端会将对应图片地址更新为该服务器地址
		res.send('http://localhost:3000/'+userId+'/'+uploadedImage.name)
	})
})

router.get('/deleteWorkPicture', function(req, res){
	var serverPrefix = 'http://localhost:1234/'
	var str = req.query.imagePath
	var filePath = path.join(__dirname + '/../work_material/' + str.replace(serverPrefix, ''))
	fs.unlink(filePath, function(err){
		if(err){
			res.send({
				errcode: 1,
				errmsg: err.message
			})
		}else{
			res.send({
				errcode: 0,
				errmsg: 'success to delete image: ' + str
			})
		}
	})
})

module.exports = router;
```

# 历史版本说明
版本号|更新日期|更新说明
|:-:|:-:|:---|
|v1.0|2019-04-18|功能参考上面“组件功能说明”|
|v1.1|2019-04-18|增加服务器上传功能：1.增加server-url属性， 2.增加form-data属性|
|v1.2|2019-04-24|修改bug: 在上传到服务器时，Add事件是在上传成功之前触发的。|
|v1.3|2019-04-24|优化服务端地址是否填写|
|v1.4|2019-04-29|优化数据双向绑定的参数：某种情况下初始化的图片数据不能展示|
|v1.5|2019-05-08|增加上传图片数量的参数：limit，默认不限制|
|v1.6|2019-05-09|修改bug:拖动图片，更新图片顺序后，新的顺序数据未同步|
|v1.7|2019-05-15|1._self.value空判断，并初始化一个数组<br>2.web上无法限制图片数量，内部做了数量检测，并提示可用数量|
|v1.8|2019-05-20|增加上传到服务器key值参数：fileKeyName，用于在服务端通过自定义key值获取该文件数据，默认为'upload-images'.|
|v1.9|2019-05-22|增加显示上传进度参数：showUploadProgress。以提示信息的方式来提示进度|
|v1.10|2019-05-23|1.修改bug:支付宝小程序不能拖动。2.调整显示样式|
|v1.11|2019-06-13|支持集成后台服务器的自定义删除图片接口，删除图片时，自动调用该接口完成后台图片文件的删除|
|v1.12|2019-07-18|修改bug:1.后退后再进入之前的图片还存在的问题，感谢602044025@qq.com提的意见。 2.单页出现滚动条时，在拖动图片时会导致页面一起拖动|
|v1.13|2019-09-25|修改bug:1.使用v-model初始化时图片不显示。 2.修改上传进度提示默认为true，避免未上传完毕就点击了提交。 3._self的全局定义移到方法内部的局部定义，修改单页面无法使用多次的问题。|