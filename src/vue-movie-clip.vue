<template>
    <div>
        <div v-if="type==='dom'" :style="{width:width+'px',height:height+'px'}">
            <img v-for="(val,key) in frames" :src="val"/>
        </div>
        <canvas :style="{width:width+'px',height:height+'px'}" v-if="type==='canvas'">

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
                type: Boolean,
                default: true
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
                type: Number,
                default: 640
            },
            height: {
                type: Number,
                default: 320
            }
        },
        data: function () {
            return {
                playing: false, lastFrame: 1, frame: 0, iv: 0, totalFrame: 0, ctx: null, canvas: null, images: []
            }
        },
        methods: {
            updateFrame: function (val) {
                if (this.$el) {
                    if (val < 1 || val > this.totalFrame) {
                        return false
                    }
                    switch (this.type) {
                        case 'dom':
                            this.$el.children[0].children[this.lastFrame - 1].style.setProperty('display', 'none');
                            this.$el.children[0].children[this.frame - 1].style.setProperty('display', 'block');
                            break;
                        case 'canvas':
                            let img = this.images[this.frame - 1];
                            this.ctx.drawImage(img, 0, 0, this.width, this.height);
                            break;
                    }
                    this.lastFrame = val;
                    this.$emit('frame', this);
                }
            },
            nextFrame: function () {

                if (this.forward === true) {
                    this.goFixFrame(this.frame + 1)
                    if (this.frame === this.totalFrame && this.loop === false) {
                        this.stop();
                    } else {
                        this.autoNext();
                    }
                } else {
                    this.goFixFrame(this.frame - 1)
                    if (this.frame === 1 && this.loop === false) {
                        this.stop();
                    } else {
                        this.autoNext();
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
                this.frame = frame;
            },
            setFrameTime: function (time) {
                if (time >= 0) {
                    this.frameTime = time;
                }
            },
            skipTo: function (frame) {
                this.goFixFrame(frame);

            },
            autoNext: function () {
                clearTimeout(this.iv);
                this.iv = setTimeout(this.nextFrame, this.frameTime);
            },
            play: function () {
                this.playing = true;
                this.nextFrame();
                this.$emit('play', this);

            },
            stop: function () {
                this.playing = false;
                clearTimeout(this.iv);
                this.$emit('stop', this);
            }
        },

        watch: {
            width: function (val, oldVal) {
                this.canvas.width = this.width;
                this.updateFrame(this.frame)
            },
            height: function (val, oldVal) {
                this.canvas.height = this.height;
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
            let self = this;
            this.totalFrame = this.frames.length;
            this.frame = this.initFrame;

            switch (this.type) {
                case 'dom':
                    for (let i = 0; i < this.$el.children[0].children.length; i++) {
                        this.$el.children[0].children[i].style.setProperty('display', 'none');
                    }
                    break;
                case 'canvas':
                    this.canvas = this.$el.children[0];
                    this.canvas.width = this.width;
                    this.canvas.height = this.height;
                    this.ctx = this.canvas.getContext("2d");
                    for (let i = 0; i < this.totalFrame; i++) {
                        let img = new Image();
                        img.src = this.frames[i];
                        this.images.push(img)
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


    }


</script>