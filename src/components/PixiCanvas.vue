<template>
  <div class="canvas-container" ref="canvasContainer">
    <canvas class="canvas" ref="canvas"></canvas>
  </div>
</template>
  
<script>
import * as PIXI from "pixi.js";
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
    this.speed = 18;
    this.displacementScale = 20;
    this.canvasScale = 1.4;
    this.width = this.$refs.canvas.clientWidth * this.canvasScale;
    this.height = this.$refs.canvas.clientHeight * this.canvasScale;
    this.dpr = window.devicePixelRatio && window.devicePixelRatio >= 2 ? 2 : 1;
    this.renderer = new PIXI.Renderer({
      view: this.$refs.canvas,
      width: this.width,
      height: this.height,
      resolution: this.dpr,
      transparent: true,
    });
    // this.renderer.autoResize = true;
    this.setScene();
  },
  beforeDestroy() {
    this.stage.removeChildren();

    this.stage.destroy(true);

    // this.$el.removeEventListener('mouseenter', this.cancelAnimate);

    // this.$el.removeEventListener('mouseleave', this.initAnimate);

    window.removeEventListener('resize', this.resize);
  },
  methods: {
    setScene() {  
      this.stage = new PIXI.Container();

      this.setFilterArea();

      this.loadImages()
        .then(() => {
          this.createDisplacementFilter();

          this.addImages();

          this.animate();

          document.body.classList.add('show');

          // this.$el.addEventListener('mouseenter', this.cancelAnimate);

          // this.$el.addEventListener('mouseleave', this.initAnimate);

          window.addEventListener('resize', this.resize);
        });
    },

    setFilterArea() {
      this.clipRect = new PIXI.Rectangle( 0, 0, this.width, this.height );

      this.stage.filterArea = this.clipRect;
    },

    loadImages() {
      return new Promise((resolve) => {
        const loader = PIXI.Loader.shared;

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
      this.displacementSprite = PIXI.Sprite.from("./assets/img/displacement.png");

      this.displacementSprite.texture.baseTexture.wrapMode = PIXI.WRAP_MODES.REPEAT;

      this.displacementFilter = new PIXI.filters.DisplacementFilter(this.displacementSprite, this.displacementScale);

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

        slide = this.setImage(slide);

        slide.alpha = i === 0 ? 1 : 0;

        this.slides[ i ] = slide;

        if (i === 0) {
          this.stage.addChild( slide );
        }
        
        i++;
      });
    },

    setImage(slide) {
      if (this.width > this.height) {
        slide.width = this.width;

        slide.height = this.width;
      } else {
        slide.width = this.height;

        slide.height = this.height;
      }

      slide.x = this.width / 2;

      slide.y = this.height / 2;

      slide.anchor.set(0.5);

      return slide;
    },

    animate() {
      this.animation = requestAnimationFrame(this.animate.bind(this));

      this.displacementSprite.x = this.delta * this.speed;

      this.displacementSprite.y = -this.delta * this.speed;

      this.delta += this.increment;

      this.renderer.render(this.stage);
    },

    initAnimate() {
      let tl = new TimelineMax();

      tl.to( this, 0.4, {
        increment: 0.1,
        ease: 'Sine.easeIn'
      });
    },

    cancelAnimate() {
      let tl = new TimelineMax();

      tl.to( this, 0.8, {
        increment: 0,
        ease: 'Sine.easeIn'
      });
    },

    resize() {
      this.width = this.$refs.canvas.clientWidth * this.canvasScale;

      this.height = this.$refs.canvas.clientHeight * this.canvasScale;

      Object.keys(this.slides).forEach(key => {
        this.slides[key] = this.setImage(this.slides[ key ])
      })

      this.setFilterArea()

      this.renderer.resize(this.width, this.height);
    },

    handleNext(currentIndex, nextIndex) {
      return new Promise((resolve) => {
        this.slides[ nextIndex ].alpha = 0;

        this.stage.addChild(this.slides[ nextIndex ]);

        let tl = new TimelineMax({
          onComplete: () => {
            this.stage.removeChild(this.slides[ currentIndex ]);
            
            resolve();
          }
        });

        tl
        .to ( this, 0.4, {
          increment: 0.52,
          ease: 'Sine.easeIn'
        }, 'begin')

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
          x: 166,
          y: 166,
          ease: 'Sine.easeIn'
        }, 'begin')

        .to( this.slides[ nextIndex ], 0.6, {
          alpha: 1,
          ease: 'Sine.easeIn'
        }, 'begin+=0.7' )

        .to( this.stage.scale, 1.2, {
          x: 1,
          y: 1,
          ease: 'Sine.easeOut'
        }, 'begin+=1.05')

        .to( this.stage, 1.2, {
          x: 0,
          ease: 'Sine.easeOut'
        }, 'begin+=1.05')

        .to( this.displacementFilter.scale, 1.2, {
          x: this.displacementScale,
          y: this.displacementScale,
          ease: 'Power1.easeOut'
        }, 'begin+=1.25')

        .to (this, 1.2, {
          increment: this.increment,
          ease: 'Power1.easeOut'
        }, 'begin+=1.25')
      })
    }
  }
}
</script>
  