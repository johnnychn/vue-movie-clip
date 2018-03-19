<template>
    <div>
        <div ref="dom" v-if="type==='dom'" :style="{width:width,height:height}">
            <img v-for="(val,key) in frames" :width="val.length>1?width:0" :height="val.length>1?height:0" v-show=""
                 :src="val"/>
        </div>
        <canvas ref="canvas" :style="{width:width,height:height}" v-if="type==='canvas'||type==='webgl'">
        </canvas>
    </div>
</template>


/*!
* vue-keyframe-animation v0.0.1 (https://github.com/johnnyGoo/vue-movieclip)
* Author: Johnny chen
*
* Copyright 2013-2016 Johnny chen
*/
<script>
    import WebGL2D from './webgl2d'

    let canvas = !!window.CanvasRenderingContext2D

    let webgl = (function () {
        try {

            var canvas = document.createElement('canvas');
            return !!(window.WebGLRenderingContext && (canvas.getContext('webgl') || canvas.getContext('experimental-webgl')));
        } catch (e) {
            return false;
        }

    })()


    function parseInt(str) {
        return Number(str.replace('px', ''));
    }


    // 注册
    export default {
        name: 'vue-movie-clip',
        // 声明 props
        props: {
            forward: {
                type: Boolean,
                default: true
            },
            frameTime: {
                type: Number,
                default: 50
            },
            loop: {
                type: Number,
                default: 1
            },
            autoPlay: {
                type: Boolean,
                default: true
            },
            initFrame: {
                type: Number,
                default: 1
            },
            type: {
                type: String,
                default: 'canvas'
            }
            ,
            frames: {
                type: Array
            }
            ,
            width: {
                type: String,
                default: '640px'
            },
            height: {
                type: String,
                default: '360px'
            },
            render: {
                type: Boolean,
                default: true
            },
            audio:{
                default:null
            }
        },
        data: function () {
            return {
                playing: false, lastFrame: 1, frame: 0, iv: 0, totalFrame: 0, ctx: null, canvas: null, images: [],
                __width: 0, __height: 0, webgl: null, program: null, __count: 0,audio_time:0,inited:false
            }
        },
        methods: {
            updateFrame: function (val) {
                if (this.inited===true&&this.$el) {
                    if (val < 1 || val > this.totalFrame) {
                        return false
                    }
                    let img
                    if (this.render) {
                        switch (this.type) {
                            case 'dom':
                                this.$el.children[0].children[this.lastFrame - 1].style.setProperty('display', 'none');
                                this.$el.children[0].children[this.frame - 1].style.setProperty('display', 'block');
                                break;
                            case 'canvas':
                                 img= this.images[this.frame - 1];
                                this.ctx.clearRect(0, 0, this.__width, this.__height);
                                if (img.complete) {
                                    if (img.height > 0) {
                                        this.ctx.drawImage(img, 0, 0, this.__width, this.__height);
                                    }
                                }
                                break;
                            case 'webgl':
                                 img = this.images[this.frame - 1];
                                this.ctx.clearRect(0, 0, this.__width, this.__height);
                                if (img.complete) {
                                    if (img.height > 0) {
                                        this.ctx.drawImage(img, 0, 0, this.__width, this.__height);
                                    }
                                }
                                break;
                        }
                        this.lastFrame = val;
                    }
                    this.$emit('frame', this);
                }
            },
            nextFrame: function () {
                if(this.inited===true) {
                    let nextframe;
                    if (this.forward === true) {
                        nextframe = this.goFixFrame(this.frame + 1)
                        if (nextframe === this.totalFrame) {
                            this.__count++
                            if (this.loop === this.__count) {
                                this.stop();
                            } else {
                                this.autoNext();
                            }
                        } else {
                            this.autoNext();
                        }


                    } else {
                        nextframe = this.goFixFrame(this.frame - 1)
                        if (nextframe === 1) {
                            this.__count++
                            if (this.loop === this.__count) {
                                this.stop();
                            } else {
                                this.autoNext();
                            }
                        } else {
                            this.autoNext();
                        }
                    }
                    if (this.audio !== null) {
                        if (Math.floor(this.audio.currentTime) !== this.audio_time) {
                            this.audio_time = Math.floor(this.audio.currentTime);
                            //console.log(this.audio_time+':'+Math.floor(this.audio_time*((1000/this.frameTime))))
                            this.frame = this.goFixFrame(Math.floor(this.audio_time * ((1000 / this.frameTime))))
                        } else {
                            this.frame = nextframe
                        }

                    } else {
                        this.frame = nextframe
                    }
                }


            },
            goFixFrame: function (frame) {

                while (frame < 1) {
                    frame = frame + this.totalFrame;
                }
                while (frame > this.totalFrame) {
                    frame = frame - this.totalFrame;
                }
                return frame
              //  this.frame = frame;
            },
            setFrameTime: function (time) {
                if (time >= 0) {
                    this.frameTime = time;
                }
            },
            skipTo: function (frame) {
                if(this.inited===true) {

                    this.frame = this.goFixFrame(frame);
                }

            },
            autoNext: function () {
                clearTimeout(this.iv);
                this.iv = setTimeout(this.nextFrame, this.frameTime);
            },
            play: function () {
                if(this.inited===true){
                this.__count = 0;
                this.playing = true;
                this.nextFrame();
                this.$emit('play', this);
                }

            },


            pause: function () {
                this.playing = false;
                clearTimeout(this.iv);
                this.$emit('pause', this);
            },
            resume: function () {
                if(this.playing===false&&this.inited===true){
                    this.playing = true;
                    this.nextFrame();
                    this.$emit('resume', this);
                }
            },

            stop: function () {
                this.playing = false;
                clearTimeout(this.iv);
                this.$emit('stop', this);
            },

            update:function () {
                this.__update_images();
            },
            __update_images:function () {
                let self = this;
                if(this.frames===null||this.frames.length===0){
                    return;
                }
                this.inited=true;
                this.totalFrame = this.frames.length;
                this.initFrame = Math.max(0, Math.min(this.initFrame, this.totalFrame));
                this.frame = this.initFrame;
                this.__width = parseInt(this.width);
                this.__height = parseInt(this.height);
                this.__count = 0

                if (this.type === 'canvas' && !canvas) {
                    this.type = 'dom'
                }
                function isString(arg) {
                    return Object.prototype.toString.call(arg) === '[object String]';
                }


                switch (this.type) {
                    case 'dom':
                        for (let i = 0; i < this.$refs.dom.children.length; i++) {
                            this.$refs.dom.children[i].style.setProperty('display', 'none');
                        }
                        break;
                    case 'canvas':
                        this.canvas = this.$refs.canvas;
                        this.canvas.width = this.__width;
                        this.canvas.height = this.__height;

                        this.ctx = this.canvas.getContext("2d");

                        for (let i = 0; i < this.totalFrame; i++) {
                            if (isString(this.frames[i])) {
                                let img = new Image();
                                img.src = this.frames[i];
                                img.crossOrigin = "Anonymous";
                                this.images.push(img)
                            } else {
                                this.images.push(this.frames[i])
                            }

                        }

                        this.images[this.initFrame - 1].onload = function () {
                            if (self.frame === self.initFrame && this.playing !== true) {
                                self.updateFrame(self.initFrame);
                            }
                        };

                        break
                    case 'webgl':
                        this.canvas = this.$refs.canvas;
                        this.canvas.width = this.__width;
                        this.canvas.height = this.__height;
                        if (this.webgl&&webgl) {
                            WebGL2D.enable(this.canvas)
                            this.ctx = this.canvas.getContext("webgl-2d")
                        } else {
                            this.ctx = this.canvas.getContext("2d");
                        }

                        for (let i = 0; i < this.totalFrame; i++) {
                            if (isString(this.frames[i])) {
                                let img = new Image();
                                img.src = this.frames[i];
                                img.crossOrigin = "Anonymous";
                                this.images.push(img)
                            } else {
                                this.images.push(this.frames[i])
                            }

                        }

                        this.images[this.initFrame - 1].onload = function () {
                            if (self.frame === self.initFrame && this.playing !== true) {
                                self.updateFrame(self.initFrame);
                            }
                        };
                        break
                }


                self.updateFrame(self.initFrame);


                //  this.updateFrame(this.frame);
                if (this.autoPlay) {
                    this.play();
                }
            }
        },

        watch: {


            width: function (val, oldVal) {
                this.__width = parseInt(val);
                this.canvas.width = this.__width;
                this.updateFrame(this.frame)
            },
            height: function (val, oldVal) {
                this.__height = parseInt(val);
                this.canvas.height = this.__height;
                this.updateFrame(this.frame)
            }, frame: function (val, oldVal) {
                if (val !== oldVal) {
                    if (val > 0 && val <= this.totalFrame) {
                        this.updateFrame(val);
                    } else {
                        return false;
                    }
                }

            }
        },

        mounted: function () {

                this.__update_images()




        }


    }


</script>