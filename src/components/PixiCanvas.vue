<template>
  <div id="canvas-container" ref="canvasContainer">
    <canvas id="slider-canvas" ref="canvas"></canvas>
  </div>
</template>

<script>
import * as PIXI from "pixi.js/lib/core";
import * as FILTER from "pixi.js/lib/filters";
import * as LOADER from "pixi.js/lib/loaders";
import { TimelineMax } from 'gsap';
import filterImg from '@/assets/img/displacement.png';
import data from '@/data';

export default {
  name: 'PixiCanvas',
  mounted() {
    this.createApp();
  },
  methods: {
    createApp() {
      PIXI.utils.skipHello(); // turn off console branding
      // PIXI.settings.SCALE_MODE = PIXI.SCALE_MODES.LINEAR;
      // PIXI.settings.PRECISION_FRAGMENT = PIXI.PRECISION.HIGH;
      this.element = document.getElementById("canvas-container");
      this.canvasWidth = this.$refs.canvas.clientWidth;
      this.canvasHeight = this.$refs.canvas.clientHeight;
      this.dpr = window.devicePixelRatio && window.devicePixelRatio >= 2 ? 2 : 1;
      this.renderer = PIXI.autoDetectRenderer(this.canvasWidth, this.canvasHeight, { transparent: true });
      this.renderer.autoResize = true;
      this.setScene();
    },
    setScene() {
      this.element.appendChild(this.renderer.view);

      this.stage = new PIXI.Container();

      this.loadImages();
    },

    loadImages() {
      const loader = LOADER.shared;

      data.slides.forEach((slide, index) => {
        loader.add(`slide-${index}`, slide.image);
      })

      this.images = [];

      loader.load( ( loader, images ) => {
        console.log(images)
        this.images = images;

        this.createDisplacementFilter()
      });
    },

    createDisplacementFilter() {
      this.dispSprite = PIXI.Sprite.fromImage('./assets/img/displacement.png');
      this.dispSprite.texture.baseTexture.wrapMode = PIXI.WRAP_MODES.REPEAT;
      // this.dispSprite.scale.y = 6;
      // this.dispSprite.scale.x = 6;
      // this.dispSprite.x = 0;
      // this.dispSprite.y = 0;
      // this.dispSprite.anchor.x = 0.5;
      // this.dispSprite.anchor.y = 0.5;

      console.dir(this.dispSprite)

      this.dispFilter = new FILTER.DisplacementFilter(this.dispSprite);

      this.dispSprite.scale.y = 6;

      this.dispSprite.scale.x = 6;

      this.stage.addChild(this.dispSprite);

      this.addSlides();
    },

    addSlides() {
      this.slides = {};

      let i = 0;

      Object.keys(this.images).forEach(key => {
        let slide = new PIXI.Sprite( this.images[key].texture );
        slide.width = this.renderer.width;
        slide.height = this.renderer.height;
        // slide.y = i === 0 ? 0 : -this.renderer.height;

        this.slides[ i ] = slide;
        this.stage.addChild( slide );
        console.dir('added')
        i++;
      });
      this.animate();
    },

    animate() {
      // console.log('animate')
      this.animation = requestAnimationFrame(this.animate.bind(this));

      this.dispSprite.x = this.count * 50;
      this.dispSprite.y = this.count * 50;

      this.count += 0.1;

      this.stage.filters = [this.dispFilter];

      this.renderer.render(this.stage);
    }
  }
}
</script>
