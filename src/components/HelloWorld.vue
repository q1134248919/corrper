<template>
  <div class="clipper-container" ref="clipper" >
    <canvas ref="canvas" id="canvas"></canvas>

    <!-- 裁剪部分 -->
    <div class="clipper-part"  >
      <div :class="{'border':!showmadal}" class='pCanvas-container'>
        <canvas ref="pCanvas" id="pCanvas"></canvas>
      </div>
    </div>

    <!-- 底部操作栏 -->
    <div class="action-bar">
      <button class="btn-cancel" @click="cancel">取消</button>
      <button class="btn-ok" @click="sure">确认</button>
    </div>

    <!-- 背景遮罩 -->
    <div class="mask "></div>

    <!-- 手势操作层 -->
    <div class="gesture-mask" ref="gesture"></div>
    <mum v-if="showmadal"></mum>
  </div>
</template>
<script>
import Mum from './Mum'
export default {
  name: "imageClipper",

  data() {
    return {
      originXDiff: 0, //裁剪canvas与原图canvas坐标原点上的差值
      originYDiff: 0,
      ctx: null,
      pCtx: null,
      actionBarHeight: 61,
      imgScouce: null, //图片路径
      $img: null,
      imgCurrentWidth: null,
      imgCurrentHeight: null,
      imgX: null, //img对于canvas的坐标
      imgY: null,
      //图片canvas宽高
      cWidth: 0,
      cHeight: 0,
      showmadal:true,
       
    };
  },
  props:{
       img:{
         type:String
       },
     width:{
       type:Number,
       default:300
     },
      height:{
        type:Number,
       default:300
      }
  },
  mounted() {
     this.imgScouce=this.img;
    this._initCanvas();
    this._initEvent();
  },
  components:{
    Mum,
  },
  methods: {
    _initCanvas(){
      let $img = new Image(),
      width=null,
      height=null;
      $img.src=this.imgScouce;
      $img.onload = () => {
        let $canvas = document.getElementById("canvas"),
        $pCanvas = document.getElementById("pCanvas"),
        clipperClientRect = this.$refs.clipper.getBoundingClientRect(),
        clipperWidth = parseInt(this.width / window.devicePixelRatio),
        clipperHeight = parseInt(this.height / window.devicePixelRatio);
      this.ctx = $canvas.getContext("2d");
      this.pCtx = $pCanvas.getContext("2d");
        width=$img.width;
        //判断clipperWidth与clipperHeight有没有超过容器值
      if (clipperWidth < 0 || clipperWidth > clipperClientRect.width) {
        clipperWidth = 300;
      }
      if (clipperHeight < 0 || clipperHeight > clipperClientRect.height) {
        clipperHeight = 300;
      }
      //判断传入的图片如果比框小的话
      if($img.width<clipperWidth+30){
        clipperWidth=clipperWidth-40;
          clipperHeight=clipperHeight-40;
      }
      //因为canvas在手机上会被放大，因此里面的内容会模糊，这里根据手机的devicePixelRatio来放大canvas，然后再通过设置css来收缩，因此关于canvas的所有值或坐标都要乘以devicePixelRatio
      $canvas.style.width = clipperClientRect.width + "px";
      $canvas.style.height = clipperClientRect.height + "px";
      $canvas.width = this._ratio(clipperClientRect.width);
      $canvas.height = this._ratio(clipperClientRect.height);

      $pCanvas.style.width = clipperWidth + "px";
      $pCanvas.style.height = clipperHeight + "px";
      $pCanvas.width = this._ratio(clipperWidth);
      $pCanvas.height = this._ratio(clipperHeight);

      //计算两个canvas原点的x y差值
      let cClientRect = $canvas.getBoundingClientRect(),
        pClientRect = $pCanvas.getBoundingClientRect();

      this.originXDiff = pClientRect.left - cClientRect.left;
      this.originYDiff = pClientRect.top - cClientRect.top;

     
      this.cWidth = cClientRect.width;
      this.cHeight = cClientRect.height;
         this._initImg($img.width, $img.height);
         this.showmadal=false;
      };
       
      $img.crossOrigin = "Anonymous";
      this.$img = $img;
      
       
    },
    _ratio(size) {
      return parseInt(window.devicePixelRatio * size);
    },
    _initEvent() {
      let $gesture = this.$refs.gesture,
        cClientRect = this.$refs.canvas.getBoundingClientRect(),
        scx = 0, 
        scy = 0,
        iX = this.imgX,
        iY = this.imgY;
      $gesture.addEventListener("touchstart", e => {
        let finger = e.touches[0];
        scx = finger.pageX;
        scy = finger.pageY;
        iX = this.imgX;
        iY = this.imgY;
      });
      $gesture.addEventListener("touchmove", e => {
        e.preventDefault();
        let f1x = e.touches[0].pageX,
          f1y = e.touches[0].pageY;
        this._drawImage(
          iX + f1x - scx,
          iY + f1y - scy,
          this.imgCurrentWidth,
          this.imgCurrentHeight
        );
      });
      $gesture.addEventListener("touchend", e => {
        let x = this.imgX,
          y = this.imgY,
          pClientRect = this.$refs.pCanvas.getBoundingClientRect();
         if (x > pClientRect.left + pClientRect.width) {
                        x = pClientRect.left
                    } else if (x + this.imgCurrentWidth < pClientRect.left) {
                        x = pClientRect.left + pClientRect.width - this.imgCurrentWidth;
                    }
               
                    if (y > pClientRect.top + pClientRect.height) {
                        y = pClientRect.top;
                    } else if (y + this.imgCurrentHeight < pClientRect.top) {
                        y = pClientRect.top + pClientRect.height - this.imgCurrentHeight;
                    }

                    if (this.imgX !== x || this.imgY !== y) {
                        this._drawImage(x, y, this.imgCurrentWidth, this.imgCurrentHeight);
                    }
      });
    },
    _drawImage(x, y, w, h) {
      this._clearCanvas();
      this.imgX = parseInt(x);
      this.imgY = parseInt(y);
      this.imgCurrentWidth = parseInt(w);
      this.imgCurrentHeight = parseInt(h);
      //更新canvas
      this.ctx.drawImage(
        this.$img,
        this._ratio(x),
        this._ratio(y),
        this._ratio(w),
        this._ratio(h)
      );

      //更新pCanvas，只需要减去两个canvas坐标原点对应的差值即可
      this.pCtx.drawImage(
        this.$img,
        this._ratio(x - this.originXDiff),
        this._ratio(y - this.originYDiff),
        this._ratio(w),
        this._ratio(h)
      );
    },
    _initImg(w, h) {
      let eW = null,
        eH = null,
        maxW = this.cWidth,
        maxH = this.cHeight - this.actionBarHeight;

      //如果图片的宽高都少于容器的宽高，则不做处理
      if (w <= maxW && h <= maxH) {
        eW = w;
        eH = h;
      } else if (w > maxW && h <= maxH) {
        eW = maxW;
        eH = parseInt((h / w) * maxW);
      } else if (w <= maxW && h > maxH) {
        eW = parseInt((w / h) * maxH);
        eH = maxH;
      } else {
        //判断是横图还是竖图
        if (h > w) {
          eW = parseInt((w / h) * maxH);
          eH = maxH;
        } else {
          eW = maxW;
          eH = parseInt((h / w) * maxW);
        }
      }

      if (eW <= maxW && eH <= maxH) {
        this._drawImage((maxW - eW) / 2, (maxH - eH) / 2, eW, eH);
      } else {
        this._initImg(eW, eH);
      }
    },
    _clearCanvas() {
      let $canvas = this.$refs.canvas,
        $pCanvas = this.$refs.pCanvas;
      $canvas.width = $canvas.width;
      $canvas.height = $canvas.height;
      $pCanvas.width = $pCanvas.width;
      $pCanvas.height = $pCanvas.height;
    },
    cancel(){
      this.$emit("cancle");
    },
    sure(){
      let  newUrl = this.$refs.pCanvas.toDataURL("image/png");
      this.$emit("sure",newUrl);
    }
  }
};
</script>
<style lang="css">
.clipper-container {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 100;
  line-height: 0;
  background-color: #000;
}
.clipper-part {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  bottom: 61px;
  z-index: 102;
}
.pCanvas-container {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
.border{
  border: 2px solid #fff;
}
.action-bar {
  box-sizing: content-box;
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  top: auto;
  z-index: 103;
  height: 60px;
  line-height: 60px;
  border-top: 1px solid rgba(256, 256, 256, 0.3);
}
button {
  display: block;
  padding: 0 15px;
  line-height: 60px;
  font-size: 16px;
  color: #fff;
  background: none;
  border: none;
  outline: 0;
}
.btn-cancel {
  float: left;
}
.btn-ok {
  float: right;
}

.mask {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 101;
  transition: opacity 500ms;
  background-color: #000;
  opacity: 0;
}
.opacity {
  opacity: 0.8;
}

.gesture-mask {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 61px;
  z-index: 103;
}
</style>

 