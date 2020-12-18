<template>
  <div class="hello">
    <canvas id="canvas" style="border: 1px solid red;" height="400" width="400"></canvas>
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
    let response = await  fetch('./UVMap.png')
    let img_data = await response.arrayBuffer()
    CanvasKitInit().then((CanvasKit) => {
      const surface = CanvasKit.MakeCanvasSurface(canvas.id);
      if (!surface) {
        console.log('Could not make surface');
        return;
      }
      const context = CanvasKit.currentContext();
      const skcanvas = surface.getCanvas();
      let img = CanvasKit.MakeImageFromEncoded(img_data)
      let paint = new CanvasKit.Paint();
      paint.setAntiAlias(true);
      paint.setColor(CanvasKit.Color(0, 0, 0, 1.0));
      paint.setStyle(CanvasKit.PaintStyle.Stroke);
      paint.setStrokeWidth(4.0);
      paint.setPathEffect(CanvasKit.PathEffect.MakeCorner(50));

      let path = new CanvasKit.Path();
      let paths = [path];
      let paints = [paint];

      function drawFrame () {
        CanvasKit.setCurrentContext(context);
        skcanvas.drawImage(img, 128, 128, paints[0]);
        for (let i = 0; i < paints.length && i < paths.length; i++) {
          skcanvas.drawPath(paths[i], paints[i]);
        }
        skcanvas.flush();
        requestAnimationFrame(drawFrame);
      }

      let hold = false;
      canvas.addEventListener('mousemove', (e) => {
        if (!e.buttons) {
          hold = false;
          return;
        }
        if (hold) {
          path.lineTo(e.offsetX, e.offsetY);
        } else {
          paint = paint.copy();
          paint.setColor(
            CanvasKit.Color(Math.random() * 255, Math.random() * 255, Math.random() * 255, Math.random() + .2));
          paints.push(paint);
          path = new CanvasKit.Path();
          paths.push(path);
          path.moveTo(e.offsetX, e.offsetY);
        }
        hold = true;
      });
      requestAnimationFrame(drawFrame);

    });
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
