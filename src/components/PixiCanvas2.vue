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
import data from '@/data';

export default {
  name: 'PixiCanvas',
  mounted() {
    PIXI.utils.skipHello(); // turn off console branding
    PIXI.settings.SCALE_MODE = PIXI.SCALE_MODES.LINEAR;
    PIXI.settings.PRECISION_FRAGMENT = PIXI.PRECISION.HIGH;

    this.increment = 0.1;
    this.delta = 0;
    // this.speed = 30;
    this.speed = 20;
    this.width = 1400;
    this.height = 1750;
    this.dpr = window.devicePixelRatio && window.devicePixelRatio >= 2 ? 2 : 1;
    this.renderer = PIXI.autoDetectRenderer( this.width, this.height, {
      view: this.$refs.canvas,
      resolution: this.dpr,
      antialias: true,
      transparent: true,
    });
    this.renderer.autoResize = true;
    this.setScene("./assets/img/art/bloom.jpg");
  },
  beforeDestroy() {
    this.stage.removeChildren();

    this.stage.destroy(true);

    this.$el.removeEventListener('mouseenter', this.cancelAnimate);

    this.$el.removeEventListener('mouseleave', this.initAnimate);
  },
  methods: {
    setScene(url) {
      this.stage = new PIXI.Container();

      // this.stage.anchor.set(0.5);

      this.clipRect = new PIXI.Rectangle( 0, 0, this.renderer.screen.width, this.renderer.screen.height );

      this.stage.filterArea = this.clipRect;

      this.loadImages()
        .then(() => {
          this.createDisplacementFilter();

          this.addImages();

          this.animate();

          document.body.classList.add('show');

          this.$el.addEventListener('mouseenter', this.cancelAnimate);

          this.$el.addEventListener('mouseleave', this.initAnimate);
        });
    },

    loadImages() {
      return new Promise((resolve) => {
        const loader = LOADER.shared;

        data.slides.forEach((slide, index) => {
          loader.add(`slide-${index}`, slide.image);
        })

        this.images = [];

        loader.load( ( loader, images ) => {
          this.images = images;

          resolve();
        });
      })
    },

    createDisplacementFilter() {
      this.displacementSprite = PIXI.Sprite.fromImage("./assets/img/displacement.png");
      // this.displacementSprite = PIXI.Sprite.fromImage("./assets/img/displacement2.jpg");

      this.displacementSprite.texture.baseTexture.wrapMode = PIXI.WRAP_MODES.REPEAT;

      this.displacementFilter = new FILTER.DisplacementFilter(this.displacementSprite, 20);

      this.displacementSprite.scale.x = 3;

      this.displacementSprite.scale.y = 3;

      this.stage.addChild(this.displacementSprite);

      this.stage.filters = [this.displacementFilter];
    },

    addImages() {
      this.slides = {};

      let i = 0;

      Object.keys(this.images).forEach(key => {
        let slide = new PIXI.Sprite( this.images[key].texture );

        slide.width = this.renderer.width / this.dpr;

        slide.height = this.renderer.height / this.dpr;

        slide.alpha = i === 0 ? 1 : 0;

        this.slides[ i ] = slide;

        if (i === 0) {
          this.stage.addChild( slide );
        }
        
        i++;
      });
    },

    animate() {
      this.animation = requestAnimationFrame(this.animate.bind(this));

      this.displacementSprite.x = this.delta * this.speed;

      this.displacementSprite.y = -this.delta * this.speed;

      this.delta += this.increment;

      this.renderer.render(this.stage);
    },

    initAnimate() {
      let tl = new TimelineMax({
        // onStart: () => {
        //   this.animate();
        // }
      });

      tl.to( this, 0.4, {
        increment: 0.1,
        ease: 'Sine.easeIn'
      });
    },

    cancelAnimate() {
      let tl = new TimelineMax({
        // onComplete: () => cancelAnimationFrame(this.animation)
      });

      tl.to( this, 0.8, {
        increment: 0,
        ease: 'Sine.easeIn'
      });
    },

    handleNext(currentIndex, nextIndex) {
      console.dir(this.stage)
      return new Promise((resolve) => {
        this.slides[ nextIndex ].alpha = 0;
        this.stage.addChild(this.slides[ nextIndex ])


        let tl = new TimelineMax({
          onComplete: () => {
            this.stage.removeChild(this.slides[ currentIndex ])
            resolve()
          }
        });

        tl
        .to ( this, 0.4, {
          // increment: 0.25,
          // increment: 0.34,
          increment: 0.35,
          // ease: 'Sine.easeIn'
        }, 'begin')

        // .to( this.slides[ currentIndex ], 0.8, {
        //   y: this.$el.clientHeight * .025,
        //   ease: 'Sine.easeIn'
        //   // ease: 'Power2.easeIn'
        // }, 'begin' )

        // .to( this.slides[ nextIndex ], 0.8, {
        //   y: this.$el.clientHeight * .025,
        //   ease: 'Sine.easeIn'
        //   // ease: 'Power2.easeIn'
        // }, 'begin' )

        // .to( this.$refs.canvas, 0.8, {
        //   y: this.$el.clientHeight * .01,
        //   ease: 'Sine.easeIn'
        // }, 'begin')

        .to( this.stage.scale, 0.8, {
          x: 1.02,
          y: 1.02,
          ease: 'Sine.easeIn'
        }, 'begin+=0.2')

        .to( this.stage, 0.8, {
          x: this.renderer.screen.width * -.01,
          ease: 'Sine.easeIn'
        }, 'begin+=0.2')

        .to( this.displacementFilter.scale, 0.7, {
          x: 140,
          y: 140,
          ease: 'Sine.easeIn'
        }, 'begin')

        .to( this.slides[ nextIndex ], 0.6, {
          alpha: 1,
          ease: 'Sine.easeIn'
        }, 'begin+=0.65' )

        // .to( this.slides[ currentIndex ], 1.2, {
        //   y: 0,
        //   ease: 'Sine.easeOut'
        // }, 'begin+=1.2' )
        // .to( this.slides[ nextIndex ], 1.2, {
        //   y: 0,
        //   ease: 'Sine.easeOut'
        // }, 'begin+=1.2' )

        // .to( this.stage, 1.2, {
        //   y: 0,
        //   ease: 'Sine.easeOut'
        // }, 'begin+=1.2' )

        // .to( this.$refs.canvas, 1.1, {
        //   scale: 1,
        //   ease: 'Power1.easeOut'
        // }, 'begin+=1.1')

        .to( this.stage.scale, 1.2, {
          x: 1,
          y: 1,
          ease: 'Sine.easeOut'
        }, 'begin+=1')

        .to( this.stage, 1.2, {
          x: 0,
          ease: 'Sine.easeOut'
        }, 'begin+=1')

        .to( this.displacementFilter.scale, 1.2, {
          x: 20,
          y: 20,
          ease: 'Power2.easeOut'
        }, 'begin+=1.2')

        .to (this, 1.2, {
          increment: this.increment,
          ease: 'Power2.easeOut'
        }, 'begin+=1.2')
      })
    }
  }
}
</script>
  