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
            frameTime: {
                type: Number,
                default: 50
            },
            loop: {
                type: Boolean,
                default: true
            },
            playing: {
                type: Boolean,
                default: true
            },
            frame: {
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
            width: {},
            height: {}
        },
        data: function () {
            return {
                lastFrame: 1, iv: 0, totalFrame: 0,ctx:null,canvas:null,images:[]
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
                            let img=this.images[this.frame - 1];
                            this.ctx.drawImage(img,0,0,this.width,this.height);
                            break;
                    }

                    this.lastFrame = val;
                    this.$emit('frame', this);
                }
            },
            nextFrame: function () {
                if (this.frame < this.totalFrame) {
                    if (this.frame === this.totalFrame - 1) {
                        if (this.loop === false) {
                            this.frame = this.frame + 1;
                            this.playing = false;


                        } else {
                            this.frame = this.frame + 1;
                            this.autoNext();
                        }
                    } else {
                        this.frame = this.frame + 1;
                        this.autoNext();
                    }

                } else {
                    if (this.loop === true) {
                        this.frame = 1;
                        this.autoNext();
                    }
                }


            },
            autoNext: function () {
                clearTimeout(this.iv);

                this.iv = setTimeout(this.nextFrame, this.frameTime);
            },
            play: function () {

                this.autoNext();
                this.$emit('play', this);

            },
            stop: function () {
                clearTimeout(this.iv);
                this.$emit('stop', this);
            }
        },

        watch: {
            width: function (val, oldVal) {
                this.canvas.width=this.width;
                this.updateFrame(this.frame)
            },
            height: function (val, oldVal) {
                this.canvas.height=this.height;
                this.updateFrame(this.frame)
            },
            playing: function (val, oldVal) {
                if (true === val) {
                    this.play();
                } else {
                    this.stop();

                }


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
            this.totalFrame = this.frames.length;
            switch (this.type) {
                case 'dom':
                    for (let i = 0; i < this.$el.children[0].children.length; i++) {
                        this.$el.children[0].children[i].style.setProperty('display', 'none');
                    }
                    break;
                case 'canvas':
                    this.canvas=this.$el.children[0];
                    this.canvas.width=this.width;
                    this.canvas.height=this.height;
                    this.ctx=this.canvas.getContext("2d");
                    for(let i=0;i<this.totalFrame;i++){
                        let img = new Image();
                        img.src=this.frames[i];
                        this.images.push(img)
                    }


                    break
            }
            this.updateFrame(this.frame);
            if (this.playing) {
                this.play();
            }


        }


    }


</script>