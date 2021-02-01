<template>
  <div class="skullify-container">
    <div class="skullify-animation" />
  </div>
</template>

<script>
import * as lottie from "lottie-web";
import spy from "cep-spy";

const isAdobe = window.__adobe_cep__ ? true : false;
const isNode =
  typeof process !== "undefined" && process.release.name === "node";

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
    folder: {
      type: String,
      default: "",
    },
    files: {
      type: Array,
      default: () => {
        return [];
      },
    },
    uniqueRollLength: {
      type: Number,
      default: 1,
    },
    speed: {
      type: Number,
      default: 1,
    },
    direction: {
      type: Number,
      default: 1,
    },
  },
  data: () => ({
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
    activeFileIndex: 0,
    activeSegmentIndex: 0,
    previousRolls: [],
  }),
  mixins: [require("../utils/IO").default],
  async mounted() {
    console.log(this.realFolderLocation);
    // console.log(this.readFiles);
    if (this.folder) this.getFileListFromFolder();
    await this.init();
  },
  watch: {
    async animationData(val, last) {
      if (JSON.stringify(val) !== JSON.stringify(last))
        await this.reconstruct(val);
    },
    options: {
      async handler(val, last) {
        if (JSON.stringify(val) !== JSON.stringify(last))
          await this.reconstruct(val);
      },
      deep: true,
    },
    speed(val, last) {
      if (val !== last) this.setSpeed(val);
    },
    direction(val, last) {
      if (val !== last) this.setDirection(val);
    },
  },
  computed: {
    markers() {
      try {
        return this.animData.animationData.markers;
      } catch (err) {
        return [];
      }
    },
    segments() {
      try {
        if (!this.animData || !this.markers.length) return [];
        let markers = this.markers.map((marker) => {
          return marker.tm;
        });
        let list = [];
        for (let i = 0; i < markers.length - 1; i++) {
          const marker = markers[i];
          const nextmarker = markers[i + 1];
          let temp = {
            forward: [marker, nextmarker],
            reverse: [nextmarker, marker],
          };
          list.push(temp);
        }
        return list;
      } catch (err) {
        return [];
      }
    },
    realDirection() {
      return this.direction ? "forward" : "backward";
    },
    realFolderLocation() {
      if (!this.folder || !this.folder.length) return null;
      let temp = "";
      if (isAdobe) {
        if (/^\.\//.test(this.folder)) {
          console.log("HAS RELATIVE PATH");
          return `${spy.path.root}/${this.folder.replace(/^\.\//, "")}`.replace(
            /\\/gm,
            "/"
          );
        } else {
          return path.resolve(this.folder);
        }
      } else return path.resolve(this.folder);
    },
  },
  methods: {
    async getFileListFromFolder() {
      if (
        this.folder &&
        this.folder.length &&
        this.realFolderLocation &&
        this.realFolderLocation.length
      ) {
        return await this.readFiles(this.realFolderLocation);
      } else if (this.files && this.files.length) {
        //
      } else return []; // This must be browser, we can't read files here
    },
    async reconstruct(val) {
      this.animData.destroy(this.name);
      if (val) await this.init();
    },
    async init() {
      try {
        this.animData = this.buildAnimation(this.animationData);
        this.$emit("load");
        if (this.opts.autoplay) this.animData.play();
        if (this.debug) console.log(this.animData);
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
    randomNum(min, max) {
      return Math.floor(Math.random() * max) + min;
    },
    shuffleFile() {},
    shuffleSegment() {},
    rollRandom(max, excludes = []) {
      let roll = this.randomNum(0, max);
      return excludes.includes(roll) ? rollRandom(max, excludes) : roll;
    },
    playSegmentChunk(value) {
      let target = this.segments[value][this.realDirection];
      this.animData.playSegments(
        this.segments[value][this.realDirection],
        true
      );
      this.play();
      console.log(value);
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
    setDirection(direction) {
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
