// TODO 将退出全屏视图和居中显示视图加到全屏界面
<template>
  <div class="vuepress-markmap">
    <!-- <button class="button" @click="screen">点击全屏显示</button> -->
    <!-- <button class="button1" @click="rescale">点击居中显示</button> -->
    <svg :id="id" ref="markmap"></svg>
  </div>
</template>

<script>
import { Transformer } from "markmap-lib";
import * as markmap from "markmap-view";

export default {
  name: "Markmap",
  props: {
    id: {
      type: String,
      required: true,
    },
    code: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      mm: null,
      flag: false,
      height: 0,
    };
  },
  mounted() {
    const transformer = new Transformer();

    // transform markdown
    const { root, features } = transformer.transform(this.code);

    // get assets required by used features
    const { styles, scripts } = transformer.getUsedAssets(features);

    const { Markmap, loadCSS, loadJS } = markmap;

    //  load assets
    if (styles) loadCSS(styles);
    if (scripts) loadJS(scripts, { getMarkmap: () => markmap });

    // create markmap
    const svg = this.$refs.markmap;
    this.mm = Markmap.create(svg, null, root);
    this.rescale();

    // rescale graph when window size changes
    const that = this;
    window.onresize = () => {
      return (() => {
        that.rescale();
      })();
    };
  },

  methods: {
    rescale() {
      // resize svg and fit graph
      // https://github.com/gera2ld/markmap/issues/63
      const svg = this.$refs.markmap;
      const that = this;
      this.mm.rescale(1).then(function () {
        svg.style.height = svg.getBBox().height + 10 + "px";
        if (svg.getBBox().height + 10 > 900) {
          svg.style.height = "900px";
        }
        that.mm.fit();
      });
    },
  },
};
</script>

<style lang="stylus">
.vuepress-markmap {
  margin: 30px 0;
  width: 100%;

  svg {
    display: block;
    width: 100%;
  }
}
</style>
