# vue-movie-clip
Sequence of frames for vue.js


```html
  <vue-movie-clip ref="my_movieclip" :auto-play="true" :init-frame="10" :width="width"
                    :height="360" type="dom" v-show="true" :frame-time="frameTime"
                    :frame="frame" :frames="frames"
                    :loop="loop" :forward="forward"
                    @frame="onFrame" @play="onPlay" @stop="onStop">
    </vue-movie-clip>
````


参数：
```
width=640
height=320
type="canvas" 
auto-play=true
frame-time="50" 
frames="frames"
loop="true"
forward="true"
```


 
 
 事件
 ```
  @frame="onFrame" //参数movieclip对象 
  @play="onPlay" 
  @stop="onStop"
 
 ```

 
 方法
 ```
 play()
 stop()
```

 
 属性
 ```
 playing
 frameTime
 forward
 width
 height
```

 
 
 
 