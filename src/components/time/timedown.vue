<template>
    <div>
            <span v-html = "str"></span>
            <span ref="tpl" v-show="false"> 
                <slot></slot>
            </span>
    </div>
</template>

<script>
export default {
        data (){
            return {
                timer:'',
                str:'',
                formats:''
            }
        },
        props:{
            // 结束后的回调
            callBack:{
                type:Function
            },
            // 结束时提示语
            endTips:{
                type:String,
                default(){
                    return '已结束'
                }
            },
            // 时间类型
            timeType:{
                validator(value){
                    return ['dateTime','second','timeStamp'].indexOf(value) > -1;
                },
                default:'timeStamp'
            },
            // 时间
            time:{
                type:[String,Number,Date]
            },
            //模板格式
            format:{
                type:String,
                default:'{d}天{h}时{m}分{s}秒'
            }

        },
        mounted(){
            this.$nextTick(() =>{
                console.log(this.$slots,this.$refs)
               this.formats = !!this.$slots.default? this.$refs.tpl.innerHTML:this.format;
               console.log(this.formats)
                this.transTime()
            })
        },
        destoryed(){
            clearInterval(this.timer);
        },
        watch:{
            time(val){
                    clearInterval(this.timer);
                    val && this.transTime();
            }
        },
        methods:{
            /* 
                时间转化(s)
            */
           timestampTotime(time){
               let formats = this.formats;
               let t = {};
               t.s = time % 60;
               time = Math.floor(time / 60);
               t.m = time % 60;
               time = Math.floor(time / 60);
               t.h = time % 24;
               t.d = Math.floor(time / 24);
               const arr = ['s','m','h','d']
               arr.forEach(v =>{
                   t[v] < 10 ? '0'+t[v] : t[v];
                //    this.formats = this.formats.replace(`{${v}}`,t[v])
               });
               formats = formats.replace('{d}',t.d);
               formats = formats.replace('{h}',t.h);
               formats = formats.replace('{m}',t.m);
               formats = formats.replace('{s}',t.s);
            //    console.log(formats)    
            return formats;
           },
           
           /* 
              计算结束时间  
           */
            leaveTime(){
                let endTime = this.lasttime - Math.floor(new Date().getTime() / 1000);
                if(endTime > 0){
                    this.str = this.timestampTotime(endTime);
                }else {
                    this.callBack&&this.callBack();
                    this.str = this.endTips;
                    //停止计算
                    clearInterval(this.timer);
                }
            },
            /* 
                转化格式，转化完毕后开始倒计时
            */
           transTime(){
               if(!this.time){
                   return
               }
               const actions = new Map();
               //传入时间为时间戳(结束时的时间戳)
               actions.set('timestamp',this.lasttime = Math.floor(new Date(this.time).getTime()/1000));
               //传入时间为秒(剩余多少秒)
               actions.set('second',this.lasttime = Math.floor(new Date().getTime()/1000)) + ~~this.time;
               //传入的为时间格式（结束时的时间）
               actions.set('dateTime',this.lasttime = Math.floor(new Date(this.time).getTime()/1000));

                actions.get(this.timeType);
                this.leaveTime();

                this.timer = setInterval(this.leaveTime,1000);
                
           }
        }
}
</script>

<style>

</style>

