<template>
  <div class="page">
    <PixiCanvas ref="canvas" />
    <SlideMaster
      :indicators="false"
      :swipe="false"
      @onSliderEvent="onSliderEvent"
      @onSliderMount="onSliderMount"
    >
      <h1 slot="slides" class="slide" v-for="(slide, index) in data.slides" ref="titles">{{ slide.title }}</h1>
    </SlideMaster>
  </div>
</template>

<script>
import data from '@/data';
import PixiCanvas from '@/components/PixiCanvas';
import SlideMaster from '@/components/SlideMaster';

export default {
  name: 'home',
  components: {
    PixiCanvas,
    SlideMaster
  },
  data() {
    return {
      data: data,
      app: null
    }
  },
  mounted() {
    setTimeout(() => this.slider.toggleEvents(), 2400);
    this.$refs.titles[0].classList.add('is-active');
  },
  methods: {
    onSliderMount(slider) {
      slider.toggleEvents(false);
      this.slider = slider;
    },

    onSliderEvent(e, slider, currentIndex, nextIndex, direction) {
      this.$refs.titles[currentIndex].classList.remove('is-active');
      this.$refs.titles[nextIndex].classList.add('is-active');
      this.$refs.canvas.handleNext(currentIndex, nextIndex)
        .then(() => this.slider.toggleEvents());
    }
  }
}
</script>
