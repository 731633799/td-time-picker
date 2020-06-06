Auth:DOUBLE Y
github:https://github.com/731633799/td-time-picker

# td-time-picker
uniapp预约时间选择器通用组件

# 使用方法

**下载组件到项目components目录**

###下载地址
https://github.com/731633799/td-time-picker

###引入组件
`import tdTimePicker from "../../components/td-time-picker/index.vue";`

###注册组件 
`components:{tdTimePicker},`

###模板中使用
`<td-time-picker></td-time-picker>`

###可选参数
####days：         可选择的周期时间/天
####step：         步进时间/分
####start_hour：   周期开始时间/小时
####start_minute： 周期开始时间/分钟
####end_hour:      周期结束时间/小时
####end_minute：   周期结束时间/分钟
####except:        不可选时间 
### @change        选择时触发 change 事件
