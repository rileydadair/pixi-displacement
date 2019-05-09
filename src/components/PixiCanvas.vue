<template>
  <div id="canvas-container" ref="canvasContainer">
    <canvas id="slider-canvas" ref="canvas"></canvas>
  </div>
</template>

<script>
import * as PIXI from 'pixi.js';
import { TimelineMax } from 'gsap';
import { Sprite } from 'pixi.js';
import filterImg from '@/assets/img/displacement.png';
import data from '@/data';

export default {
  name: 'PixiCanvas',
  mounted() {
    this.createApp();
    this.loadImages();
  },
  methods: {
    createApp() {
      PIXI.utils.skipHello(); // turn off console branding
      PIXI.settings.SCALE_MODE = PIXI.SCALE_MODES.LINEAR;
      PIXI.settings.PRECISION_FRAGMENT = PIXI.PRECISION.HIGH;

      this.canvasWidth = this.$refs.canvas.clientWidth;
      this.canvasHeight = this.$refs.canvas.clientWidth;
      
      this.app = new PIXI.Application({
        view: this.$refs.canvas,
        width: this.canvasWidth,
        height: this.canvasHeight,
        resolution: window.devicePixelRatio && window.devicePixelRatio >= 2 ? 2 : 1,
        transparent: true,
        // autoResize: true
      });
    },

    loadImages() {
      const loader = PIXI.Loader.shared;

      data.slides.forEach((slide, index) => {
        loader.add(`slide-${index}`, slide.image);
      })

      this.images = [];

      loader.load( ( loader, images ) => {
        // images are multi-dimensional objects
        this.images = images;
        this.createSlider();
        // this.loader.hide();
      });
    },

    createSlider() {
      this.slider = new PIXI.Container();
      this.slider.width = this.app.screen.width;
      this.slider.height = this.app.screen.height;
      this.app.stage.addChild( this.slider );

      this.clipRect = new PIXI.Rectangle( 0, 0, this.app.screen.width, this.app.screen.height );
      this.slider.filterArea = this.clipRect;

      this.addSlides();
      this.createDisplacementFilter();
    },

    addSlides() {
      this.slides = {};

      let i = 0;

      Object.keys(this.images).forEach(key => {
        let slide = new PIXI.Sprite( this.images[key].texture );
        slide.width = this.app.screen.width;
        slide.height = this.app.screen.height;
        slide.y = i === 0 ? 0 : -this.app.screen.height;

        this.slides[ i ] = slide;
        this.slider.addChild( slide );

        i++;
      });
    },

    createDisplacementFilter() {
      this.dispSprite = PIXI.Sprite.from('./assets/img/displacement.png');
      this.dispSprite.texture.baseTexture.wrapMode = PIXI.WRAP_MODES.REPEAT;
      // this.dispSprite.scale.y = 1.8;
      // this.dispSprite.scale.x = 1.8;

      // this.app.stage.addChild( this.dispSprite );

      this.dispFilter = new PIXI.filters.DisplacementFilter(this.dispSprite, 0);

      // this.slider.filters = [ this.dispFilter ];


      this.app.stage.filters = [this.dispFilter];
      this.app.stage.addChild(this.dispSprite);

      this.speed = .5;
      this.animateFilter();
    },

    animateFilter() {
      this.dispSprite.x += this.speed;
      this.dispSprite.y += this.speed;
      // this.app.renderer.render(this.stage);
      requestAnimationFrame(this.animateFilter);

      // const tl = new TimelineMax({ repeat: -1, repeatDelay: 0 })
      //   .to( this.dispFilter.scale, 1, {
      //     x: 5,
      //     y: 5,
      //     // ease: 'Power2.easeInOut'
      //   }, 0 )

      //   .to( this.dispFilter.scale, 1, {
      //     x: 0,
      //     y: 0,
      //     // ease: 'Power2.easeInOut'
      //   }, 1 )
    }
  }
}
</script>
