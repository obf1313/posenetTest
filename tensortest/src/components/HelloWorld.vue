<template>
  <div class="hello">
    <img id="picture" src="../assets/000161.jpg" style="display: none;" />
    <canvas id="canvas"></canvas>
  </div>
</template>

<script>
  import * as posenet from '@tensorflow-models/posenet';
  export default {
    name: 'HelloWorld',
    data() {
      return {
        pointList: []
      }
    },
    methods: {
      estimatePoseOnImage: async function estimatePoseOnImage(imageElement) {
        // imageScaleFactor 介于0.2-1.0之间的数字，图像传输之前需要缩放，将此数值设置得更低，以缩小图像的比例，并在牺牲精度的情况下提高网络传输速度
        // flipHorizontal 默认 false，如果姿势应该水平翻转/镜像，对于视频默认水平翻转的情况，应该将其设置为true
        // outputStride 通过模型输入图像时输出所需的步幅,32,16,8，默认16，数字越大，性能越快，但准确性越低，反之亦然
        const imageScaleFactor = 0.2;
        const flipHorizontal = false;
        const outputStride = 16;
        // load the posenet model from a checkpoint
        const net = await posenet.load();

        const pose = await net.estimateSinglePose(imageElement, imageScaleFactor, flipHorizontal, outputStride);

        // 它返回一个带有置信度得分的姿势和一个由part id索引的关键点数组，每个关键点都有一个得分和位置
        return pose;
      },
      getCTX: function (pointList,imageElement) {
        let c = document.getElementById("canvas");
        let ctx = c.getContext("2d");
        c.width = imageElement.width;
        c.height = imageElement.height;
        ctx.drawImage(imageElement,0,0);
        for(let i=0; i<pointList.length; i++) {
          let point = pointList[i];
          this.drawPoint(ctx,point);
        }
      },
      drawPoint: function (ctx,point) {
        ctx.fillStyle = "#00f6ff";
        ctx.fillRect(point.x,point.y,20,20);
        // ctx.font = "50px bold 宋体";
        // ctx.fillText("("+point.x.toString().substring(0,2)+","+point.y.toString().substring(0,2)+")",point.x,point.y)
      },
      getPointList: function (keypoints,imageElement) {
        let pointList = [];
        for(let i=0; i<keypoints.length; i++) {
          pointList.push(keypoints[i].position);
        }
        this.getCTX(pointList,imageElement)
      }
    },
    mounted() {
      const imageElement = document.getElementById('picture');
      const pose = this.estimatePoseOnImage(imageElement);
      pose.then((result) => {
        this.getPointList(result.keypoints,imageElement);
      });
    }
  }
</script>

<style scoped>
</style>
