<template>
    <div>
        <el-dialog id="dialog" title="拍照" :visible.sync="dialogVisible" width="60%">
            <div class="videoArea">
                <video src="" id="videoEl" ref="videoRef" @mousemove="video_mousemove">

                </video>
                <!-- 水印 -->
                <div ref="watermark" class="watermark" @mousedown="watermark_mouserdown" @mouseup="watermark_mouseup"
                    :style="{color:styles.color? styles.color:'#8c00d8',fontSize:styles.fontSize? styles.fontSize:'16px',fontFamily:styles.fontFamily?styles.fontFamily:'fantasy'}">
                    <div>{{sign}}</div>
                    <div>{{time}}</div>
                    <div v-html="html" ref="html"></div>
                </div>
            </div>
            <span slot="footer" class="dialog-footer">
                <el-input v-model="sign"></el-input>
                <el-button @click="dialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="confirm">确 定</el-button>
            </span>
            <el-dialog width="50%" title="内层 Dialog" :visible.sync="innerVisible"  append-to-body>
                <div>
                    <img :src="URL" alt="">
                </div>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="dialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="confirm">确 定</el-button>
                </span>
            </el-dialog>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: 'ahjetDialog',
        data() {
            return {
                STREAM: null,
                dialogVisible: false,
                // 客户输入的文字
                sign: '默认文字',
                time: '',
                address: '',
                mediaOPtions: {
                    video: true,
                    audio: false
                },
                html: '',
                // 标识是否被按下
                isclickDown: false,
                // 自定义参数
                styles: {
                    color: '',
                    fontSize: '',
                    fontFamily: ''
                },
                innerVisible:false,
                URL:''
            };
        },
        mounted() {
            this.close()
            // 申请获取用户设备的摄像头
            navigator.mediaDevices.getUserMedia(this.mediaOPtions).
                then((stream) => {
                    this.STREAM = stream
                    this.$refs.videoRef.srcObject = stream
                    this.$refs.videoRef.play()
                }).
                catch((err) => {
                    this.$message.error('已拒绝访问设备相机');
                })
            // 获取当前时间
            this.time = new Date().toLocaleDateString() + '' + new Date().toLocaleTimeString()
            // 获取定位
            navigator.geolocation.getCurrentPosition((res) => {
                console.log(res)
            }, (err) => {
                console.log(err)
            })
            // console.log(this.address)
        },
        methods: {
            open() {
                this.dialogVisible = true
            },
            confirm() {

                // 530 x 400
                let canvas = document.createElement('canvas')
                canvas.width = 530
                canvas.height = 400
                let ctx = canvas.getContext('2d')
                ctx.drawImage(this.$refs.videoRef, 0, 0, 530, 400, 0, 0, 530, 400)
                ctx.fillStyle = this.styles.color || '#8c00d8'
                
                
                ctx.font = `normal ${this.styles.fontSize ? this.styles.fontSize : '16px'} ${this.styles.fontFamily ? this.styles.fontFamily : 'fantasy'}`;
                ctx.fillText(this.sign, parseInt(this.$refs.watermark.style.left), parseInt(this.$refs.watermark.style.top))
                ctx.fillText(this.time, parseInt(this.$refs.watermark.style.left), parseInt(this.$refs.watermark.style.top)+15)
                var url = canvas.toDataURL("image/png");
                console.log(url)
                this.URL = url
                this.STREAM.getVideoTracks().forEach(track => {

                    track.stop()

                });
                this.$refs.videoRef.srcObject = null
                this.innerVisible = true
            },
            close() {
                this.dialogVisible = false
                if (this.STREAM) {
                    this.STREAM.getVideoTracks().forEach(track => {

                        track.stop()

                    });
                    this.$refs.videoRef.srcObject = null
                }
            },
            watermark_mouserdown() {
                this.isclickDown = true
            },
            watermark_mouseup() {
                this.isclickDown = false
            },
            video_mousemove(e) {
                if (this.isclickDown) {
                    this.$refs.watermark.style.top = e.offsetY + 'px'
                    this.$refs.watermark.style.left = (e.offsetX - 63) + 'px'
                }
            }
        }
    }
</script>
<style>
    #dialog {
        height: 100%;
        /* position: relative; */
    }

    #videoEl {
        width: 530px;
        height: 400px;
        /* transform: rotateY(180deg); */
    }

    .el-dialog__body {
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .dialog-footer {
        display: flex;
    }

    .watermark {
        position: absolute;
        top: 0;
        left: 0;
        width: 126px;
        cursor: pointer;
        user-select: none;
        /* transform: rotateY(180deg); */
    }

    .videoArea {
        position: relative;
        width: 530px;
        height: 400px;
        display: flex;
        align-items: center;
        justify-content: center;
    }
</style>