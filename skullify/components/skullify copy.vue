<template>
  <div class="skullify-container">
    <div class="skullify-animation" />
  </div>
</template>

<script>
import * as lottie from "lottie-web";

export default {
  name: "skullify",
  props: {
    animationData: {
      type: Object,
      default: () => {
        return null;
      },
    },
    debug: {
      type: Boolean,
      default: false,
    },
    name: {
      type: String,
      default: () => {
        return (
          Date.now().toString(36) + Math.random().toString(36).substring(2)
        );
      },
    },
    options: {
      type: Object,
      default: () => {
        return {
          loop: false,
          prerender: true,
          autoplay: false,
          animType: "svg",
        };
      },
    },
  },
  data: () => ({
    speed: 1,
    elt: null,
    doneLoading: false,
    animData: null,
    autoplay: true,
    override: false,
    lottieElt: null,
    activeItem: null,
    isDragging: false,
    loop: true,
    opts: {
      loop: false,
      prerender: true,
      autoplay: false,
      animType: "svg",
    },
  }),
  async mounted() {
    await this.init();
    console.log(this.name);
  },
  watch: {
    async animationData(val) {
      await this.reconstruct(val);
    },
    options: {
      async handler(val) {
        await this.reconstruct(val);
      },
      deep: true,
    },
  },
  methods: {
    async reconstruct(val) {
      this.animData.destroy(this.name);
      if (val) await this.init();
    },
    async init() {
      try {
        this.animData = this.buildAnimation(this.animationData);
        this.$emit("load");
        if (this.opts.autoplay) this.animData.play();
        console.log(this.animData);
        return Promise.resolve(true);
      } catch (err) {
        return Promise.reject(err);
      }
    },
    buildAnimation(anim = null) {
      if (!anim) anim = this.animationData;
      return lottie.loadAnimation(
        Object.assign(
          {
            animationData: anim,
            wrapper: this.$el.children[0],
            name: this.name,
          },
          Object.assign(this.opts, this.options)
        )
      );
    },
    play() {
      this.animData.play();
    },
    stop() {
      this.animData.stop();
    },
    goToAndStop(value, isFrame) {
      this.animData.goToAndStop(value, isFrame, this.name);
    },
    setSpeed(speed) {
      this.animData.setSpeed(speed, this.name);
    },
    setSpeed(direction) {
      this.animData.setDirection(direction, this.name);
    },
    freeze() {
      this.animData.freeze();
    },
    unfreeze() {
      this.animData.unfreeze();
    },
    inBrowser() {
      return this.animData.inBrowser();
    },
    resize() {
      this.animData.resize();
    },
    setQuality(quality) {
      this.animData.setQuality(quality);
    },
  },
};
</script>

<style>
.skullify-container svg {
  width: 100%;
}
</style>
