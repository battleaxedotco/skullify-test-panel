<template>
  <div id="app">
    <Menus refresh debug />
    <Panel>
      <Wrapper>
        <Button label="Roll index" @click="shuffle" />
        <Input-Scroll
          label="File index"
          v-model="activeIndex"
          :min="0"
          :max="files.length - 1"
        />
      </Wrapper>
      <div class="lottie-wrapper">
        <Anno>{{ currentTitle }}</Anno>
        <skullify
          debug
          folder="./src/assets/wrenches"
          :animationData="currentAnimationData"
          :options="{
            loop: true,
          }"
        />
      </div>
      <Divider alt />
      <div v-if="false">
        <Wrapper>
          <Button label="Roll segment index" @click="shuffleSegment" />
          <Input-Scroll
            label="Segment index"
            v-model="activeSegmentIndex"
            :min="0"
            :max="3"
          />
        </Wrapper>
        <div class="lottie-wrapper">
          <Anno>{{ currentTitle }}</Anno>
          <skullify
            ref="ball"
            :animationData="ballAnimation"
            :options="{
              loop: true,
            }"
          />
        </div>
      </div>
    </Panel>
  </div>
</template>

<script>
import { skullify } from "../skullify";
import { readDir, readFile } from "arcanophile";
import spy from "cep-spy";

export default {
  data: () => ({
    activeIndex: 0,
    activeSegmentIndex: 0,
    files: [],
    fileNames: [],
    ballAnimation: require("../src/assets/ball.json"),
  }),
  components: {
    skullify,
  },
  computed: {
    currentAnimationData(val) {
      return this.files.length ? this.files[this.activeIndex] : null;
    },
    currentTitle() {
      return this.fileNames[this.activeIndex];
    },
  },
  watch: {
    currentAnimationData(val, last) {
      if (JSON.stringify(val) !== JSON.stringify(last)) {
        console.log("ANIMATION CHANGED:");
        console.log(val);
      }
    },
  },
  async mounted() {
    await this.getShuffleFiles();
  },
  methods: {
    async getShuffleFiles() {
      const root = `${spy.path.root}/src/assets/wrenches`;
      for (let file of await readDir(root)) {
        this.files.push(JSON.parse(await readFile(`${root}/${file}`, false)));
        this.fileNames.push(file.replace(".json", ""));
      }
      return true;
    },
    randomNum(min, max) {
      return Math.floor(Math.random() * max) + min;
    },
    getRandomIndex(max = null) {
      let last = this.activeIndex;
      let roll = this.randomNum(0, this.files.length);
      return roll !== last ? roll : this.getRandomIndex(max);
    },
    shuffle() {
      this.activeIndex = this.getRandomIndex(this.files.length);
    },
    shuffleSegment() {
      this.activeSegmentIndex = this.getRandomIndex(4);
      this.$refs.ball.playSegmentChunk(this.activeSegmentIndex);
    },
  },
};
</script>

<style>
.lottie-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  padding: 20px 20%;
  flex-direction: column;
  box-sizing: border-box;
}

.anim-main,
.anim-color {
  fill: var(--color-default);
  stroke: var(--color-default);
}

.anim-mask {
  fill: var(--color-bg);
  stroke: var(--color-bg);
}
.anim-bg {
  fill: transparent;
  stroke: transparent;
}
</style>
