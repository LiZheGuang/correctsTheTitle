<template lang="pug">
.correcting(v-if="commitImageUrls")
    .correcting-view(v-if="isPageView === 'view'")
      .topNavTool
        el-button.el-button-fs.arrow( type="text" icon="el-icon-arrow-left" @click="clickUpNewImageView")
        span 第{{pathImagsIndex  + 1}}张 / 共{{pathImags.length}}张
        el-button.el-button-fs.arrow( type="text" icon="el-icon-arrow-right" @click="clickGoNewImageView")
      .maskFather-box()
        img.viewImage( :src="pathImags[pathImagsIndex].answer")
    .correcting-edit(v-if="isPageView !== 'view'")
      .topNavTool()
          el-button(type="text" @click="clickCircle" v-if="isPageView !== 'view'") 
              img.elButtonImage( type="text" src="https://wx-static.yangcong345.com/answer_radir.png")
          el-button.el-button-fs.error(  type="text" icon="el-icon-close" @click="clickImage('close')")  
          el-button.el-button-fs.check(  type="text" icon="el-icon-check" @click="clickImage('check')")   
          el-button.el-button-fs.error(  type="text" icon="el-icon-edit-outline" @click="clickPushText")  
          el-button.el-button-fs.inintColor(  type="text" icon="el-icon-delete" @click="clickDelete")   
          el-button.el-button-fs.inintColor(  type="text" icon="el-icon-refresh" @click="clickAllCanvas")    
          el-button.el-button-fs.inintColor(  type="text" icon="el-icon-refresh-right" @click="rotate(90)")
          el-button.el-button-fs.arrow( type="text" icon="el-icon-arrow-left" @click="clickUpNewImage")
          span 第{{pathImagsIndex  + 1}}张 / 共{{pathImags.length}}张
          el-button.el-button-fs.arrow( type="text" icon="el-icon-arrow-right" @click="clickGoNewImage")
      .maskFather-box()
        canvas#mask()
      //-   img.demoImg(:src="item.base64" v-for="item of pathImags" v-if="item.base64")
</template>

<script>
import { fabric } from "fabric";
export default {
  name: "correcting",
  props: ["commitImageUrls", "isPageView"],
  data() {
    return {
      canvas: "",
      rect: "",
      base64: "",
      pathImagsIndex: 0,
      pathImags: [
        // {
        //   answer:
        //     "https://wx-static.yangcong345.com/CDD4DA7A-C834-453C-9848-45970A300A92-15856278439799026.png?imageMogr2/thumbnail/750x",
        //   loadFromJSON: "", //用来存储数据
        //   base64: "",
        //   // 是否增加了背景图
        //   bgcImg: false,
        //   direction: 0
        // },
        // {
        //   answer:
        //     "https://wx-static.yangcong345.com//h5/promo-intro-v7/promo-sku-6-5.png?imageMogr2/thumbnail/750x",
        //   loadFromJSON: "", //用来存储数据
        //   base64: "",
        //   bgcImg: false,
        //   direction: 0
        // },
        // {
        //   answer:
        //     "https://wx-static.yangcong345.com/CDD4DA7A-C834-453C-9848-45970A300A92-15856278439799026.png?imageMogr2/thumbnail/750x",
        //   loadFromJSON: "", //用来存储数据
        //   base64: "",
        //   bgcImg: false,
        //   direction: 0
        // },
        // {
        //   answer:
        //     "https://wx-static.yangcong345.com//h5/promo-intro-v7/promo-sku-6-5.png?imageMogr2/thumbnail/750x",
        //   loadFromJSON: "", //用来存储数据
        //   base64: "",
        //   bgcImg: false,
        //   direction: 0
        // }
      ],
      outPathImage: []
    };
  },
  watch: {
    // 监测传递参数，进行处理
    commitImageUrls(newVal) {
      if (newVal) {
        let pathImags = [];
        newVal.map(item => {
          pathImags.push({
            answer:
              item.indexOf("imageMogr2") >= 0
                ? item
                : item + "?imageMogr2/auto-orient/thumbnail/750x/", //图片750并恢复原形旋转
            loadFromJSON: "", //用来存储数据
            base64: "",
            // 是否增加了背景图
            bgcImg: false,
            direction: 0
          });
        });
        this.pathImags = JSON.parse(JSON.stringify(pathImags));
        // 此处用来存储默认数据

        this.outPathImage = JSON.parse(JSON.stringify(pathImags));
        this.pathImagsIndex = 0;

        if (this.isPageView !== "view" && !this.canvas) {
          this.onlCanvas();
        }

        if (this.canvas && this.isPageView !== "view") {
          this.allRemove();
        }
      }
    }
  },
  methods: {
    // 预览下的上一页
    clickUpNewImageView() {
      this.pathImagsAtts("up", "view");
    },
    // 预览下的下一页
    clickGoNewImageView() {
      this.pathImagsAtts("next", "view");
    },
    //   上传图片至七牛
    upImageQiniu() {
      // 判断当前处于第几个Index,把当前index重新生成base64
      let pathImagsIndex = this.pathImagsIndex;
      let outObj = this.pathImags[pathImagsIndex];
      outObj.base64 = this.canvas.toDataURL("image/jpeg");
      this.$set(this.pathImags, pathImagsIndex, outObj);
      // 如果没有进行批改的Base64会为空，进行业务比对，把未处理图片继续返回就行
      return this.pathImags;
    },
    // 批改中的上一个
    clickUpNewImage() {
      if (this.pathImagsIndex == 0) {
        return;
      }
      this.setCanvasJson();
      this.pathImagsAtts("up");
      this.canvas.clear();
      this.imageOnl();
      console.log(this.canvas.getActiveObjects());
      console.log("上一张");
    },
    // 下一个批改的作业
    clickGoNewImage() {
      // 跳转前序列号存入data
      if (this.pathImagsIndex == this.pathImags.length - 1) {
        return;
      }
      this.setCanvasJson();
      this.pathImagsAtts("next");
      this.canvas.clear();
      this.imageOnl();
      // this.canvas.add(this.rect);
      console.log(this.canvas.getActiveObjects());
    },
    // 获取image加载完毕
    getImageRes() {
      let image = new Image();
      let pathUrl = this.pathImags[this.pathImagsIndex].answer;
      image.src = pathUrl;
      image.crossOrigin = "";
      return new Promise(resove => {
        image.onload = res => {
          console.log(res);
          resove(image);
        };
      });
    },
    // 旋转操作
    rotate() {
      let pathImagsIndex = this.pathImagsIndex;
      let outObj = this.pathImags[pathImagsIndex];
      if (this.pathImags[this.pathImagsIndex].direction >= 3) {
        outObj.direction = 0;
        this.pathImags[this.pathImagsIndex].direction = 0;
      } else {
        outObj.direction++;
      }
      this.$set(this.pathImags, pathImagsIndex, outObj);

      let direction = this.pathImags[this.pathImagsIndex].direction;
      console.log(direction);
      let canvas = this.canvas;
      const items = canvas.getObjects();
      console.log(items);
      const backImage = items.find(item => item.type === "image");
      console.log(backImage);
      if (!backImage) return;
      const { width = 800, height = 800 } = backImage;
      const translate = (height - width) / 2;
      const matrixMap = {
        0: [1.0, 0.0, 0.0, 1.0, 0, 0],
        1: [0.0, 1.0, -1.0, 0.0, translate, -translate],
        2: [-1.0, 0.0, -0.0, -1.0, 0, 0, 0, 0],
        3: [-0.0, -1.0, 1.0, -0.0, translate, -translate]
      };

      backImage.transformMatrix = matrixMap[direction];
      canvas.setWidth(direction % 2 ? height : width);
      canvas.setHeight(direction % 2 ? width : height);
      canvas.requestRenderAll();
    },
    imageOnl() {
      let image = new Image();
      let createCanvas = document.createElement("canvas");
      let ctx = createCanvas.getContext("2d");

      let pathUrl = this.pathImags[this.pathImagsIndex].answer;
      image.src = pathUrl;
      image.crossOrigin = "";

      image.onload = res => {
        console.log(res);
        console.log("图片加载完毕");
        console.log(image.width);
        console.log(image.height);
        ctx.drawImage(image, 0, 0, image.width, image.height);
        // pathUrl = createCanvas.toDataURL("image/jpeg");
        this.base64 = pathUrl;
        this.canvas.width = image.width;
        this.canvas.height = image.height;
        if (this.pathImags[this.pathImagsIndex].bgcImg === false) {
          fabric.Image.fromURL(
            pathUrl,
            oImg => {
              oImg.selectable = false;

              this.canvas.add(oImg);
              this.canvas.setHeight(oImg.height || "800");
              this.canvas.setWidth(oImg.width || "800");
              let pathImagsIndex = this.pathImagsIndex;
              let outObj = this.pathImags[pathImagsIndex];
              outObj.bgcImg = true;
              //   翻页的时候生成上一页的base64
              this.$set(this.pathImags, pathImagsIndex, outObj);
              console.log(this.canvas);
              console.log(this.canvas.getObjects());
            },
            { crossOrigin: "Anonymous" }
          );
        } else {
          // console.log(this.canvas);
          // console.log(this.canvas.getObjects());
          this.canvas.getObjects()[0].selectable = false;
          let direction = this.pathImags[this.pathImagsIndex].direction;
          let items = this.canvas.getObjects();
          console.log(items);
          const backImage = items.find(item => item.type === "image");
          const { width = 800, height = 800 } = backImage;
          const translate = (height - width) / 2;
          const matrixMap = {
            0: [1.0, 0.0, 0.0, 1.0, 0, 0],
            1: [0.0, 1.0, -1.0, 0.0, translate, -translate],
            2: [-1.0, 0.0, -0.0, -1.0, 0, 0, 0, 0],
            3: [-0.0, -1.0, 1.0, -0.0, translate, -translate]
          };

          backImage.transformMatrix = matrixMap[direction];
          this.canvas.setWidth(direction % 2 ? height : width);
          this.canvas.setHeight(direction % 2 ? width : height);
          // console.log(this.direction);
          console.log("背景图已经被缓存过");
        }
      };
    },
    // 删除所有方法
    allRemove() {
      this.pathImags = JSON.parse(JSON.stringify(this.outPathImage));
      this.pathImagsIndex = 0;
      var objects = this.canvas.getObjects();
      for (var i = 0; i < objects.length; i++) {
        this.canvas.remove(objects[i]);
      }
      this.canvas.renderAll();
      this.imageOnl();
    },
    // 点击清除所有元素
    clickAllCanvas() {
      // 此处是个撤销
      // var canvasObjects = this.canvas.getObjects();
      // this.canvas.remove(canvasObjects[0]);
      // 此处是个撤销
      this.$confirm(
        "确定要重置图片吗?重置后图片将恢复到用户提交的初始状态",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消"
        }
      )
        .then(() => {
          // this.canvas.clear();
          this.pathImags = JSON.parse(JSON.stringify(this.outPathImage));

          // this.canvas = "";
          // this.rect = "";
          this.pathImagsIndex = 0;
          var objects = this.canvas.getObjects();
          for (var i = 0; i < objects.length; i++) {
            //console.log(objects[i]);
            this.canvas.remove(objects[i]);
          }
          console.log("296");
          this.canvas.renderAll();
          // this.onlCanvas();
          this.imageOnl();
        })

        .catch(() => {});
    },
    // 点击图片
    clickImage(typeName) {
      let pathRequire = {
        check: require("@/assets/anser_yes.png"),
        close: require("@/assets/answer_error.png")
      };
      fabric.Image.fromURL(pathRequire[typeName], oImg => {
        // scale image down, and flip it, before adding it onto canvas
        oImg.scale(0.5).set("flipX, true");
        this.canvas.add(oImg);
      });

      console.log(this.canvas.getObjects());
    },
    // 点击增加文案
    clickPushText() {
      var text = new fabric.Textbox("请输入文案", {
        fontFamily: "Regular-Regular",
        left: 100,
        top: 100,
        fontSize: 40,
        fill: "#E63131"
      });
      this.canvas.add(text);
    },
    // 画一个圈
    clickCircle() {
      this.rect = new fabric.Circle({
        radius: 65,
        fill: "transparent",
        left: 110,
        opacity: 0.7,
        stroke: "red",
        strokeWidth: 5,
        strokeUniform: true
      });
      this.canvas.add(this.rect);
    },
    // 删除
    clickDelete() {
      let activeObject = this.canvas.getActiveObject();
      // console.log(activeObject);
      let getActiveObjects = this.canvas.getActiveObjects();
      console.log(activeObject);
      console.log(getActiveObjects);
      if (getActiveObjects.length > 0) {
        getActiveObjects.forEach(item => {
          console.log(item);
          this.canvas.remove(item);
        });
      } else {
        this.canvas.remove(activeObject);
      }
    },
    // 圆
    clickPushMask() {
      this.rect = new fabric.Rect({
        left: Math.ceil(Math.random() * 100), //距离画布左侧的距离，单位是像素

        top: Math.ceil(Math.random() * 200), //距离画布上边的距离

        fill: "red", //填充的颜色

        width: 30, //方形的宽度

        height: 30, //方形的高度,
        selectable: true
      });
      this.canvas.add(this.rect);
    },
    // 画布初始化
    async onlCanvas() {
       await this.getImageRes();
      this.canvas = new fabric.Canvas("mask", {
        // backgroundColor: "#ebebeb",
        // skipTargetFind: true,
        width: 750,
        height: 750
      });
      // console.log(this.canvas)
      this.imageOnl();
      // this.canvas.add(this.rect);

      // 组件监听
      this.canvas.on("object:selected", function(options) {
        console.log(options);
        console.log(options.e.clientX, options.e.clientY);
      });
    },
    // 上下图片逻辑
    pathImagsAtts(name, type) {
      // 来获取总过有多少张图片
      let pathImagsAttsLen = this.pathImags.length;
      // 当前是第几张图片
      if (name === "next") {
        console.log(this.pathImagsAtts.length);
        console.log(this.pathImagsIndex);
        if (this.pathImagsIndex + 1 >= pathImagsAttsLen) {
          this.pathImagsIndex = pathImagsAttsLen - 1;
        } else {
          this.pathImagsIndex++;
        }
      } else {
        if (this.pathImagsIndex <= 0) {
          this.pathImagsIndex = 0;
        } else {
          this.pathImagsIndex--;
        }
      }
      if (!type) {
        // 如果下一页有序列化，把序列化的内容展现出来
        if (this.pathImags[this.pathImagsIndex].loadFromJSON) {
          this.canvas.loadFromJSON(
            this.pathImags[this.pathImagsIndex].loadFromJSON
          );
        }
      }
    },

    setCanvasJson() {
      let pathImagsIndex = this.pathImagsIndex;
      // 把当前的cnavas序列化
      let jsonString = JSON.stringify(this.canvas);
      let outObj = this.pathImags[pathImagsIndex];
      console.log(outObj);
      outObj.loadFromJSON = jsonString;
      //   翻页的时候生成上一页的base64
      outObj.base64 = this.canvas.toDataURL("image/jpeg");
      this.$set(this.pathImags, pathImagsIndex, outObj);
    }
  },
  mounted() {}
};
</script>

<style lang="stylus" scoped>

.correcting
  // width 780px
  border solid 1px #ebebeb
  border-radius 5px
  padding-top 0
  padding-bottom 20px
  // display flex
  // align-items center
  // flex-wrap wrap
  // justify-content center
  .maskFather-box
    width 100%
    overflow scroll
    .viewImage
      width 750px
  .topNavTool
    width 100%
    box-sizing border-box
    padding 6px 8px
    // padding-bottom 12px
    border-bottom 1px solid #ebebeb
    display flex
    align-items center
    .el-button-fs
      font-size 24px
      margin-right 6px
    .elButtonImage
      width 24px
      height 24px
      cursor pointer
    .check
      color #14af5a
    .error
      color #d81e06
    .arrow
      color #373737
  #mask
    margin 10px auto
  .demoImg
    width 250px
</style>
