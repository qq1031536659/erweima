<template>
    <div>
        <button class='share_quan' open-type="getUserInfo" @getuserinfo='share_quan'>分享朋友圈</button>
         <button class="share" type="primary" open-type="getUserInfo" @getuserinfo="getUserInfo">开始答题（获取用户信息）</button>
        <!-- 画布 -->
        <canvas  canvas-id="shareImg" style="width:100%;height:600px;margin:0 auto"></canvas> -->
        <div :hidden='hidden'> </div>   
        <image :src='imgurl'></image>
        <button  @click='save'>保存图片到相册</button>
        <!-- <div class="shareQR">
            <img src="" alt="">
            <p>转发至群或好友，好友打开后，即可绑定上下级关系</p>
        </div>
        <div class="downLoad">
            <img src="" alt="">
            <p>打开图片二维码，长按保存至手机，即可分享朋友圈，好友打开后，即可绑定上下级关系</p>
        </div> -->
    </div>
</template>
<script>
    export default {
        data(){
            return{
                hidden: true,
                code: "/static/image/zm.png",
                title: "",
                imgs:"/static/image/5.jpg",
                imgUrl:"",
                windowWidth: '',
                posterHeight: '',
                avatarUrl:''
            }
        },
        methods:{
            getUserInfo(e){
               console.log(e)
            },
            // canvas绘制文字和图片
            setCanvas(){
                const poster = {
                    "with": 375,
                    "height": 587
                }
                const ctx = wx.createCanvasContext('shareImg');
                const systemInfo = wx.getSystemInfoSync()
                let windowWidth = systemInfo.windowWidth
                let windowHeight = systemInfo.windowHeight
                let posterHeight = parseInt((windowWidth / poster.with) * poster.height)
                
                ctx.drawImage(this.imgs, 0, 0, windowWidth, posterHeight);  //绘制背景图
                const qrImgSize = 120
                ctx.drawImage(this.code, ( windowWidth - qrImgSize) / 2, 290, qrImgSize, qrImgSize) //绘制二维码
                ctx.drawImage(this.avatarUrl, 31, 31, 60, 60) //绘制头像
                ctx.setTextAlign('center');                        
                ctx.setFillStyle('black');                    
                ctx.setFontSize(28);                              
                ctx.setFillStyle('gray'); 
                ctx.fillText("长按二维码....", 600/2, 740);
        
                ctx.stroke();
                ctx.draw();
            },
            //画布生成图片
            share_quan(e){ //点击分享朋友圈，画布生成图片   
                this.avatarUrl=e.target.userInfo.avatarUrl;
                var that = this;
                wx.showLoading({
                    title: '图片正在生成中...'
                })
                this.setCanvas()
                setTimeout( ()=> {  //这里要加定时器，转成图片需要一定的时间，不然是不出来图片的哦
                // canvas画布转成图片
                    wx.canvasToTempFilePath({
                        x: 0,
                        y: 0,
                        width: 600,
                        height: 800,
                        destWidth: 480,
                        destHeight: 550,
                        canvasId: 'shareImg',
                        fileType: 'jpg' ,  //这里为图片格式，最好改为jpg，如果png的话，图片存到手机可能有黑色背景部分
                        success: function (res) {
                            that.imgurl=res.tempFilePath,
                            that.hidden=false    
                            wx.hideLoading()
                        },
                        fail: function () {
                            console.log("保存失败......")
                        }
                    })
                }, 2000)
            },
            //点击保存到相册
            save(){
                wx.saveImageToPhotosAlbum({
                    filePath:this.imgurl,
                    success:(res)=> {
                        wx.showModal({
                            content: '图片已保存到相册',
                            showCancel: false,
                            confirmText: '好的',
                            success:(res)=> {
                                if (res.confirm) {
                                    console.log('用户确定了');
                                    this.hidden=true
                                }
                            }
                        })
                    }
                })
              
            },
        },
        mounted(){
            
        },
        onLoad(options){
            wx.setNavigationBarTitle({ title: '我的二维码' });
        }
    }
</script>
<style lang="scss" scoped>
    .shareQR,.downLoad{
        width:85%;
        margin:0 auto;
        img{

        }
        p{
            font-size:16px;
            font-family:Arial, Helvetica, sans-serif;
        }
    }
</style>>