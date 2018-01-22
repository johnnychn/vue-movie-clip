# vue-movie-clip
Sequence of frames for vue.js


```html
  <vue-movie-clip ref="my_movieclip" :auto-play="true" :init-frame="10" width="640px"
                             height="360px" type="canvas" v-show="true" :frame-time="frameTime"
                             :frame="frame" :frames="frames"
                             :loop="loop" :forward="forward"
                             @frame="onFrame" @play="onPlay" @stop="onStop">
             </vue-movie-clip>
````
###Usage
```
npm install vue-movie-clip --save
```
```javascript
import VueMovieClip from 'vue-movie-clip'
```


###props:
```
width:String "640px"/"*px"
height:String "360px"/"*px"
type:String "canvas"/"dom"
autoPlay:Boolean true/false
frameTime:Number 50/(1+) 
frames:Array []/['xxx.jpg']
loop:Boolean true/false
forward:Boolean true/false
```

###methods
 ```
 play()
 stop()
 skipTo(frame:Number)
 setFrameTime(frameTime:Number)
```
 
 
###Events
 ```
frame 
play
stop
 
 ```


###properties
 ```
 playing:Boolean
 frameTime:Number
 forward:Boolean
 width:String
 height:String
```

 
 
 
 