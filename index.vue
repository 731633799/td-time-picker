<template>
	<view class="timePicker">
		<scroll-view class="left" scroll-y="true">
			<view :class="{item:true,active:index==active_date}" @click="chose_day(index)" v-for="(item,index) in timeDate" :key="index">
				{{item.date}} <text v-if="item.lable">（{{item.lable}}）</text>
			</view>
		</scroll-view>
		<scroll-view class="right" scroll-y="true" @scroll="scroll" :scroll-top="scrollTop">
			<view :class="{'item':true,'disable':item.disable,'active':active_time_index==index}" v-for="(item,index) in chose_data" :key="index" @click="chose_time(index)">
				<view class="time">
					{{item.start_time}} <text v-if="item.end_time">-{{item.end_time}}</text>
				</view>
				<view class="tip"  v-if="item.tip">
					<text >{{item.tip}}</text>
				</view>
			</view>
		</scroll-view>
	</view>
</template>

<script>
    export default {
        name:"tdTimePicker",
        props:{
            dateData:{
                type:Array ,
                default:null
            },
            days:{
                type:Number,
                default:7
            },
            step:{
                type:Number,
                default:30
            },
            start_hour:{
                type:Number,
                default:9
            },
            start_minute:{
                type:[Number,String],
                default:0
            },
            end_hour:{
                type:Number,
                default:21
            },
            end_minute:{
                type:Number,
                default:0
            },
            except:{
                type:Array,
                default:null
            }
        },
        data() {
            return {
                scrollTop:0,
                active_date:0,
                active_time_index:-1,
            };
        },
        computed: {
            timeDate(){
                return this.dateData?this.dateData:this.create_data();
            },
            chose_data() {
                return  (this.timeDate && this.timeDate[this.active_date])?this.timeDate[this.active_date].data:[];
            }
        },
        methods: {
            chose_day(index){
                this.active_date=index;
                this.scrollTop = 0;
                this.active_time_index=-1;
            },
            scroll: function(e) {
                this.scrollTop= e.detail.scrollTop
            },
            chose_time(index){
                let temp=this.chose_data[index];
                if(temp.disable!=true){
                    this.active_time_index=index;
                    const temp_data=this.timeDate[this.active_date];
                    const time_data=temp_data.data[this.active_time_index];
                    let var_temp_data={...temp_data};
                    delete var_temp_data.data
                    this.$emit("change",{date:var_temp_data,time:time_data})
                }
            },
            create_data(){
                let date_len=this.days;
                let setup=this.step;
                let start_hour=this.start_hour;
                let start_minute=this.start_minute;
                let end_hour=this.end_hour;
                let end_minute=this.end_minute;
                let date=new Date();
                date.setHours(start_hour);
                date.setMinutes(start_minute);
                date.setSeconds(0);
                let timestamp=Date.parse(date)/1000; //当前时间戳
                var arr=[];
                const week_data= ["周日","周一","周二","周三","周四","周五","周六"]
                for(let i_data=0;i_data<date_len;i_data++){
                    let new_timestamp= parseInt(timestamp)+(3600*24*i_data);
                    let new_date= new Date(new_timestamp*1000);//每次开始的时间
                    let lable_day=	new_date.getDate()<10?"0"+new_date.getDate():new_date.getDate();
                    let lable_month=new_date.getMonth()<10?"0"+(parseInt(new_date.getMonth())+1):new_date.getMonth()+1;
                    let lable=week_data[new_date.getDay()]
                    if(i_data==0){
                        lable="今天";
                    }else if(i_data==1){
                        lable="明天"
                    }else if(i_data==2){
                        lable="后天"
                    }
                    arr[i_data]={lable:lable,year:new_date.getFullYear(),date:[lable_month,lable_day].join("-"),data:[]};
                    new_date.setHours(end_hour);
                    new_date.setMinutes(end_minute);
                    new_date.setSeconds(0);
                    let end_timestamp=(Date.parse(new_date))/1000; //每次结束时间
                    while(new_timestamp<=end_timestamp){
                        let temp_new_timestamp=new_timestamp;
                        let start_temp_date=new Date(new_timestamp*1000);
                        start_temp_date.setSeconds(0);
                        new_timestamp=new_timestamp+60*(setup<=0?60:setup);
                        let end_temp_date=new Date(new_timestamp*1000);
                        end_temp_date.setSeconds(0);
                        if(setup!=0 && Date.parse(end_temp_date)/1000>end_timestamp){
                            continue;
                        }
                        let temp_start_hour=start_temp_date.getHours();
                        let temp_start_minute=start_temp_date.getMinutes();
                        let var_temp_start_hour=temp_start_hour<10?"0"+temp_start_hour:temp_start_hour;
                        let var_temp_start_minute=temp_start_minute<10?"0"+temp_start_minute:temp_start_minute;
                        let start_time=[var_temp_start_hour,var_temp_start_minute].join(":");
                        let temp_end_hour=end_temp_date.getHours();
                        let temp_end_minute=end_temp_date.getMinutes();
                        let var_temp_end_hour=temp_end_hour<10?"0"+temp_end_hour:temp_end_hour;
                        let var_temp_end_minute=temp_end_minute<10?"0"+temp_end_minute:temp_end_minute;
                        let end_time=setup<=0?'':[var_temp_end_hour,var_temp_end_minute].join(":")
                        let disable=Date.parse(new Date())/1000+(setup>0?setup*60:0)>Date.parse(new Date(start_temp_date))/1000?true:false;
                        let tip='';
                        for(let index in this.except){
                            let temp=this.except[index];
                            let temp_start_time_date=new Date(temp.start_time);
                            temp_start_time_date.setSeconds(0)
                            let temp_end_time_date=new Date(temp.end_time);
                            temp_end_time_date.setSeconds(0);
                            if(temp_new_timestamp>=Date.parse(temp_start_time_date)/1000 && new_timestamp<=Date.parse(temp_end_time_date)/1000){
                                tip=temp.tip;
                                disable=true;
                            }
                        }
                        let temp_item={
                            start_time:start_time,
                            end_time:end_time,
                            tip:tip,
                            disable:disable
                        };
                        arr[i_data].data.push(temp_item)
                    }
                }
                return arr;
            }
        },
    }
</script>
<style scoped lang="scss">
	$activeColor:red;
	.timePicker {
		background-color: #fff;
		height: 100%;
		min-height: 200upx;
		display: flex;
		.left,.right{
			height: inherit;
		}
		.left{
			width: 240upx;
			background-color: #ECECEC;
			text-align: center;
			font-size: 30upx;
			.item{
				&.active{
					position: relative;
					color: $activeColor;
					background-color: #fff;
					&::before{
						position: absolute;
						left: 0;
						content: '';
						width: 10upx;
						background-color: $activeColor;
						height: 100%;
						display: inline-block;
					}
				}
				box-sizing: border-box;
				height: 90upx;
				line-height: 90upx;
			}
		}
		.right{
			flex: 1;
			display: flex;
			align-items: center;
			justify-content: space-between;
			.item{
				width: 50%;
				box-sizing: border-box;
				padding-top: 30upx;
				text-align: center;
				display: inline-flex;
				flex-direction: column;
				.time{
					font-size: 27upx;
					margin-bottom: 20upx;
				}
				.tip{
					font-size: 24upx;
					color: #dedede;
					min-height: 30upx;
				}
				&.disable{
					color: #dedede;
				}
				&.active{
					color: $activeColor;
				}
			}
		}
	}
</style>
