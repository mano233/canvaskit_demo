<template>
  <div class="hello">
    <img id="test_img" width="100" height="100"/>
    <canvas id="canvas" style="border: 1px solid red;" height="800" width="800"></canvas>
    <canvas id="canvas1" style="border: 1px solid green;" height="800" width="800"></canvas>
  </div>
</template>

<script>
import CanvasKitInit from 'canvaskit-wasm/bin/canvaskit'

export default {
  name: 'HelloWorld',
  data () {
    return {}
  },
  async mounted () {
    let canvas = document.getElementById('canvas')
    // let response = await fetch('https://img.iplaysoft.com/wp-content/uploads/2019/free-images/free_stock_photo.jpg')
    let bg_img_data = await fetch('./static/bg.jpg').then((response) => {
      return response.arrayBuffer()
    })
    let fr_img_data = await fetch('./static/fr.png').then((response) => {
      return response.arrayBuffer()
    })
    CanvasKitInit().then((CanvasKit) => {
      const surface = CanvasKit.MakeWebGLCanvasSurface('canvas')
      const surface1 = CanvasKit.MakeWebGLCanvasSurface('canvas1')
      if (!surface) {
        console.log('Could not make surface');
        return;
      }
      // const context = CanvasKit.currentContext();
      // const skcanvas = surface.getCanvas();
      // const skcanvas1 = surface1.getCanvas();
      // encode img
      let bg_img = CanvasKit.MakeImageFromEncoded(bg_img_data)
      let fr_img = CanvasKit.MakeImageFromEncoded(fr_img_data)
      let paint = new CanvasKit.Paint()
      paint.setAntiAlias(true)
      paint.setColor(CanvasKit.Color(0, 0, 0, 1.0))
      paint.setStyle(CanvasKit.PaintStyle.Stroke)
      paint.setStrokeWidth(40.0)
      paint.setPathEffect(CanvasKit.PathEffect.MakeCorner(50))

      let path = new CanvasKit.Path();
      let paths = [];

      // 保存一下原始的混合模式
      let baseBlendMode = paint.getBlendMode()
      let paint1 = paint.copy()
      function drawFrame (canvas) {
        // CanvasKit.setCurrentContext(context);
        canvas.drawImage(bg_img, 0, 0, paint)
        let layer_0_id = canvas.saveLayer()
        let layer_1_id = canvas.save()

        canvas.scale(0.1, 0.1)
        canvas.drawImage(fr_img, 0, 0, paint)
        canvas.restoreToCount(layer_1_id)
        paint1.setColor(CanvasKit.Color(255, 0, 255, 1));
        // 绘制所有的笔刷
        for (let i = 0; i < paths.length; i++) {
          // 根据每个笔刷的混合模式来绘制（每次设置有性能损耗，应该可以继续优化，我笔记本风扇在咆哮）
          paint1.setBlendMode(paths[i].blend)
          canvas.drawPath(paths[i].path, paint1)
        }
        canvas.restoreToCount(layer_0_id)
        canvas.flush();
      }
      function drawFrame1(canvas){
        canvas.drawColor(CanvasKit.Color(255,255,0,1))
        canvas.drawImage(CanvasKit.MakeImageFromCanvasImageSource(document.getElementById('canvas')),0,0,paint)
        document.getElementById('test_img').src = document.getElementById('canvas').toDataURL()
        canvas.flush();
      }

      let hold = false;
      canvas.addEventListener('mousemove', (e) => {
        if (!e.buttons) {
          hold = false
          return
        }
        if (hold) {
          path.lineTo(e.offsetX, e.offsetY);
        } else {
          path = new CanvasKit.Path();
          paths.push({path,blend:baseBlendMode});
          path.moveTo(e.offsetX, e.offsetY);
        }
        hold = true;
        surface.requestAnimationFrame(drawFrame);

      });
      canvas.addEventListener('mouseup',(e)=>{
        // 鼠标抬起的时候，修改最后一条笔刷的混合模式
        paths[paths.length-1].blend = CanvasKit.BlendMode.SrcATop
        surface.requestAnimationFrame(drawFrame);
        surface1.requestAnimationFrame(drawFrame1)
      })
      surface.requestAnimationFrame(drawFrame);
      surface1.requestAnimationFrame(drawFrame1)
    });
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
