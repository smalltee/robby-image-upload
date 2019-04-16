# 项目说明
本项目是基于uni-app的一个通用组件。可支持小程序，也支持APP。
组件名称为“标签组件”，通过属性设置可支持只读标签，也支持可编辑标签

# 组件功能说明：
1. 输入文字，点添加，可新建一个标签
2. 可同时输入多个标签，以逗号相隔
3. 标签新建后自动添加到后面
4. 标签上自带删除图标，点击可删除
5. 删除图标可配置显示/隐藏
6. 标签背景色可配置内置的5种颜色
7. 标签可点击触发事件
8. 标签删除时可触发删除事件
9. 添加区域可配置显示/隐藏
10. 可获取当前标签的数据列表

# 使用说明
拷贝该组件到components目录下之后

在 `script`中引用组件
```
import robbyTags from '@/components/robby-tags/robby-tags.vue'
export default {
    components: {robbyTags}
}
```

在 `template` 中使用组件：
```
<robby-tags></robby-tags>
<robby-tags @add="addTag" @delete="delTag" :tag-data="tagData" :enable-del="enableDel" :enable-add="enableAdd"></robby-tags>
```

# 属性说明
属性名|类型|默认值|说明
:-:|:-:|:-:|-
enable-del|Boolean|false|删除图标是否可见，即是否可删除
enable-add|Boolean|false|添加标签操作是否可见，即是否可进行添加标签
tag-data|Array<String>|[]|初始化的标签数据
bg-color-type|String|''|标签的背景颜色，默认为灰色，另外可取值有四种，与uni自带的颜色保持一致，分别为：primary, success, warn, error

# 事件说明
事件名称|说明|返回参数
:-:|-|-
add|点击”添加“按钮后触发的事件,返回参数为当前操作添加的标签数组和当前所有的标签数组|{currentTag:Array<String>,allTags:<String>}
delete|点击“x”删除图标后触发的事件,返回参数为当前删除的标签字符串|{currentTag:String,allTags:<String>}
click|点击标签文字触发的事件，返回参数为当前点击的标签字符串|String

# demo示例
```
<template>
	<view class="uni-common-mt">
		<view class="uni-label">
			为作品贴标签
		</view>
		<robby-tags @add="addTag" @delete="delTag" @click="clickTag" :tag-data="tagList" :enable-del="enableDel" :enable-add="enableAdd"></robby-tags>
	</view>
</template>
<script>
	import robbyTags from '@/components/robby-tags/robby-tags.vue'
	export default {
		data() {
			return {
				enableDel: true,
				enableAdd: true,
				colorType: 'primary',
				tagList: ['建筑','动漫','艺术']
			}
		},
		methods:{
			clickTag: function(e){
				console.log(e)
			},
			delTag: function(e){
				console.log(e)
			},
			addTag: function(e){
				console.log(e)
			}
		}
	}
</script>
```