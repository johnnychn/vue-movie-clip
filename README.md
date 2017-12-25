# vue-movie-clip
Sequence of frames for vue.js


```html
 <vue-movie-clip :width="width" :height="360" type="canvas" v-show="true" :playing="playing" :frame-time="50"
                    :frame="frame" :frames="frames"
                    :loop="true" @frame="onFrame" @play="onPlay" @stop="onStop">
    </vue-movie-clip>
````


参数：

width 
height
type="canvas" 
playing=
frame-time="50" 
frame="frame" 
frames="frames"
loop="true"
 
 
 
 事件
 @frame="onFrame" 参数movieclip对象 
 @play="onPlay" 
 @stop="onStop"
 