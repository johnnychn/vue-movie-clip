<template>
    <div>
        <slot></slot>
    </div>
</template>


/*!
* vue-keyframe-animation v0.0.1 (https://github.com/johnnyGoo/vue-movieclip)
* Author: Johnny chen
*
* Copyright 2013-2016 Johnny chen
*/
<script>


    var count = 0;

    if (!window.Smart) {
        throw 'vue-keyframe-animation required smart.js'
    }
    var Smart = window.Smart;
    var Css = Smart.Css;
    var _ = Smart._;

    var show = {display: 'block'};
    var hide = {display: 'none'};

    // 注册
    export default {
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
            }
        },
        data: function () {
            return {
                lastFrame: 1, iv: 0, totalFrame: 0
            }

        },
        methods: {
            updateFrame: function (val) {
                if (val < 1 || val > this.totalFrame) {
                    return false
                }
                var visible = {};
                this.frame = val;
                Css.css(this.$el.children[this.lastFrame - 1], hide);
                Css.css(this.$el.children[this.frame - 1], show);
                this.lastFrame = val;
                this.$emit('frame', this);
            },
            nextFrame: function () {

                if (this.frame == this.totalFrame) {
                    if (this.loop == false) {
                        this.stop();
                        return;
                    } else {
                        this.updateFrame(1);
                    }

                } else {
                    this.updateFrame(this.frame + 1);
                }

                if (this.frame == this.totalFrame) {
                    if (this.loop == false) {
                        this.stop();
                    }
                }
                if (this.playing) {
                    this.play();
                }
            },
            play: function () {
                clearTimeout(this.iv);
                this.playing = true;
                this.iv = setTimeout(this.nextFrame, this.frameTime);


            },
            stop: function () {
                clearTimeout(this.iv);
                this.playing = false;
                this.$emit('stop', this);
            }
        },

        watch: {
            playing: function (val, oldVal) {

              if(val){
                  this.play();
                  this.$emit('play', this);
              }else{
                  this.stop();
              }

            }
        },
        computed: {},


        ready: function () {
            _.each(this.$el.children, function (el) {
                Css.smartCss(el, hide);
            });
            this.totalFrame = this.$el.children.length;
            this.updateFrame(this.frame);

            if (this.playing) {
                this.play();
            }

        }


    }


</script>