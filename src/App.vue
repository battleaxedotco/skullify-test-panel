<template>
  <div id="app">
    <Menus refresh debug />
    <Panel>
      <div style="max-height: 50%">
        <Grid template="repeat(2, 1fr)">
          <!-- FOLDER PATH -->
          <div class="anim-wrapper">
            <Anno>Folder prop:</Anno>
            <skullify
              ref="shuffle1"
              folder="./src/assets/wrenches"
              :options="{
                autoplay: true,
              }"
            />
            <Button block label="Roll index" @click="shuffle(1)" />
          </div>

          <!-- FILE PATHS -->
          <div class="anim-wrapper">
            <Anno>Filepath array:</Anno>
            <skullify
              ref="shuffle2"
              :files="[
                './src/assets/wrenches/white.json',
                './src/assets/wrenches/blue.json',
                './src/assets/wrenches/green.json',
                './src/assets/wrenches/yellow.json',
              ]"
              :uniqueRollLength="2"
              :options="{
                autoplay: true,
              }"
            />
            <Button block label="Roll index" @click="shuffle(2)" />
          </div>

          <!-- FILE DATA (LOTTIE JSON) ARRAY -->
          <div class="anim-wrapper">
            <Anno>File data array:</Anno>
            <skullify
              ref="shuffle3"
              :files="fileList"
              :options="{
                autoplay: true,
              }"
            />
            <Button block label="Roll index" @click="shuffle(3)" />
          </div>

          <!-- DIRECT LOTTIE PROP REPLACEMENT -->
          <div class="anim-wrapper">
            <Anno>Animation data prop:</Anno>
            <skullify
              ref="shuffle4"
              :animationData="currentAnimationData"
              :options="{
                autoplay: true,
              }"
            />
            <Button block label="Roll index" @click="shuffleDirect" />
          </div>
        </Grid>
      </div>
    </Panel>
  </div>
</template>

<script>
import { skullify } from "../skullify";

export default {
  data: () => ({
    activeIndex: 0,
    activeSegmentIndex: 0,
    fileList: [
      require("./assets/wrenches/blue.json"),
      require("./assets/wrenches/green.json"),
      require("./assets/wrenches/white.json"),
      require("./assets/wrenches/yellow.json"),
    ],
    ballAnimation: require("../src/assets/ball.json"),
  }),
  components: {
    skullify,
  },
  computed: {
    currentAnimationData(val) {
      return this.fileList.length ? this.fileList[this.activeIndex] : null;
    },
  },
  methods: {
    shuffle(num) {
      this.$refs[`shuffle${num}`].shuffleFile();
    },

    // Everything below is for #4 only:
    shuffleDirect() {
      this.activeIndex = this.rollRandom(this.fileList.length, [
        this.activeIndex,
      ]);
      console.log(this.activeIndex);
    },
    randomNum(min, max) {
      return Math.floor(Math.random() * max) + min;
    },
    rollRandom(max, excludes = []) {
      let roll = this.randomNum(0, max);
      return excludes.includes(roll) ? this.rollRandom(max, excludes) : roll;
    },
  },
};
</script>

<style>
.anim-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  padding: 20px 20%;
  flex-direction: column;
  box-sizing: border-box;
  border: 2px solid var(--color-input-border);
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
