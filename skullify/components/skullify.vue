<template>
  <div class="skullify-container">
    <div class="skullify-animation" />
  </div>
</template>

<script>
import * as lottie from "lottie-web";
import spy from "cep-spy";
import path from "path";

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
    animData: null,
    opts: {
      loop: false,
      prerender: true,
      autoplay: false,
      animType: "svg",
    },
    realFiles: [],
    activeFileIndex: 0,
    activeSegmentIndex: 0,
    previousFileRolls: [],
    previousSegmentRolls: [],
    loaded: false,
    mounted: false,
  }),
  mixins: [require("../utils/IO").default, require("../utils/lottie").default],
  async mounted() {
    if (this.debug) console.log(this.realFolderLocation);
    // if (this.debug) console.log(this.readFiles);
    if (this.folder) {
      if (this.debug) console.log("FOLDER:");
      this.realFiles = await this.getFileListFromFolder();
    } else if (this.files && this.files.length) {
      if (this.debug) console.log("FILES:");
      this.realFiles = await this.parseFileList();
      if (this.debug) console.log("DONE PARSING");
    }
    await this.init();
  },
  watch: {
    loaded(val) {
      this.$emit(`${val ? "" : "un"}load`);
    },
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
    realFiles(val) {
      if (this.debug) console.log("FILE LIST CHANGED:");
      if (this.debug) console.log(val);
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
      return this.sanitizeLocalPath(this.folder);
    },
  },
  methods: {
    async parseFileList() {
      let temp = [];
      for (let file of this.files.filter((file) => {
        return /string/i.test(typeof file) ? /(json|lottie)$/.test(file) : true;
      }))
        if (/string/i.test(typeof file))
          temp.push(
            JSON.parse(await this.readFile(this.sanitizeLocalPath(file), false))
          );
        else if (/object/i.test(typeof file)) temp.push(file);
        else this.$emit("error", `${typeof file} is not supported`);
      return temp.filter((item) => {
        return this.checkIfValidLottie(item);
      });
    },
    checkIfValidLottie(item) {
      return (
        Object.keys(item).length &&
        ["v", "fr", "ip"].some((el) => Object.keys(item).includes(el))
      );
    },
    sanitizeLocalPath(targ) {
      return /^\.\//.test(targ) && isAdobe
        ? `${spy.path.root}/${targ.replace(/^\.\//, "")}`.replace(/\\/gm, "/")
        : path.resolve(targ);
    },
    async getFileListFromFolder() {
      if (this.debug) console.log("READING FROM FOLDER");
      if (
        this.folder &&
        this.folder.length &&
        this.realFolderLocation &&
        this.realFolderLocation.length
      ) {
        let temp = await this.readFiles(this.realFolderLocation, false);
        return temp
          .filter((item) => {
            return /string/i.test(typeof item);
          })
          .map((item) => {
            return JSON.parse(item);
          })
          .filter((item) => {
            return this.checkIfValidLottie(item);
          });
      } else if (this.files && this.files.length) {
        // This should have been handled via "files" prop
      } else return []; // This must be browser, we can't read files here
    },
    async reconstruct(val) {
      this.animData.destroy();
      this.loaded = false;
      if (val) await this.init();
    },
    async init() {
      if (this.debug)
        console.log("INITIALIZING", this.animationData, this.animData);
      try {
        let temp = this.animationData
          ? this.animationData
          : this.realFiles.length
          ? this.realFiles[this.activeFileIndex]
          : null;

        if (!temp) {
          this.$emit("error", "Animation data was empty");
          return null;
        } else this.animData = this.buildAnimation(temp);

        if (this.opts.autoplay && this.animData && this.animData.play)
          this.animData.play();
        if (this.debug) console.log(this.animData);
        this.loaded = true;
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
    shuffleFile() {
      while (this.previousFileRolls.length > this.uniqueRollLength)
        this.previousFileRolls.shift();
      let roll = this.rollRandom(this.realFiles.length, this.previousFileRolls);
      this.previousFileRolls.push(roll);
      this.activeFileIndex = roll;
      this.reconstruct(true);
    },
    shuffleSegment() {},
    rollRandom(max, excludes = []) {
      let roll = this.randomNum(0, max);
      return excludes.includes(roll) ? this.rollRandom(max, excludes) : roll;
    },
    playSegmentChunk(value) {
      this.animData.playSegments(
        this.segments[value][this.realDirection],
        true
      );
      this.play();
    },
  },
};
</script>

<style>
.skullify-container svg {
  width: 100%;
}
</style>
