<template>
  <div :class="options.namespace + '-base-container scroller-at-' + options.scrollerPosition">
    <img
      :src="previewImg.url"
      :data-zoom="previewLargeImg.url"
      class="responsive-image preview-box"
      draggable="false"
    >
    <div class="thumb-list">
      <img
        @click="moveThumbs('backward')"
        :src="scrollerIconFirst"
        class="zoomer-control responsive-image"
        alt="move thumb icon"
      >
      <img
        @mouseover="chooseThumb(thumb, $event)"
        draggable="false"
        v-show="key < options.scrollItems"
        :key="key"
        :src="thumb.url"
        @click="chooseThumb(thumb, $event)"
        v-for="(thumb, key) in thumbs"
        class="responsive-image"
        v-bind:style="{'boxShadow' : thumb.id === choosedThumb.id ? '0px 0px 0px 2px ' + options.choosedThumbBorderColor : ''}"
        :class="{'choosed-thumb': thumb.id === choosedThumb.id}"
      >
      <img
        @click="moveThumbs('forward')"
        :src="scrollerIconSecond"
        class="zoomer-control responsive-image"
        alt="move thumb icon"
      >
    </div>
    <div :id="paneContainerId" class="pane-container"></div>
  </div>
</template>

<script>
import Drift from "../assets/drift-zoom/src/js/Drift.js";
import FillDown from "../assets/svg-icons/arrow-down-s-fill.svg";
import LineDown from "../assets/svg-icons/arrow-down-s-line.svg";
import FillLeft from "../assets/svg-icons/arrow-left-s-fill.svg";
import LineLeft from "../assets/svg-icons/arrow-left-s-line.svg";
import FillRight from "../assets/svg-icons/arrow-right-s-fill.svg";
import LineRight from "../assets/svg-icons/arrow-right-s-line.svg";
import FillUp from "../assets/svg-icons/arrow-up-s-fill.svg";
import LineUp from "../assets/svg-icons/arrow-up-s-line.svg";
const actionName = s =>
  "scrollerAt" + (s.substr(0, 1).toUpperCase() + s.substr(1).toLowerCase());

const getCaculatedPanePosition = (paneStyle = "pane", rect, PanePosition) => {
  let caculatedPosition = "";
  if (PanePosition === "left") {
    caculatedPosition =
      "width:" +
      rect.width +
      "px;height:" +
      rect.height +
      "px;left:" + (paneStyle === "container" ? 0 : (0 - rect.width - window.scrollX - 5)) +
      "px;";
  } else if (PanePosition === "right") {
    caculatedPosition =
      "width:" +
      rect.width +
      "px;height:" +
      rect.height +
      "px;left:" + (paneStyle === "container" ? 0 : (rect.width + window.scrollX + 5)) +
      "px;";
  }

  return caculatedPosition;
}

export default {
  name: "ProductZoomer",
  props: {
    baseZoomerOptions: {
      type: Object,
      default: function() {
        return {};
      }
    },
    baseImages: {
      type: Object,
      required: true,
      default: function() {
        return {};
      }
    }
  },
  data() {
    return {
      previewImg: {},
      previewLargeImg: {},
      thumbs: [],
      normalSize: [],
      largeSize: [],
      choosedThumb: {},
      drift: null,
      options: {
        zoomFactor: 4,
        pane: "container",
        hoverDelay: 300,
        namespace: "container-zoomer",
        moveByClick: true,
        scrollItems: 4,
        choosedThumbBorderColor: "#ff3d00",
        scrollerButtonStyle: "line",
        scrollerPosition: "left",
        zoomerPanePosition: "right"
      }
    };
  },
  computed: {
    baseContainerDiv: function() {
      return document.querySelector(
        "." + this.options.namespace + "-base-container"
      );
    },
    paneContainerId: function() {
      return this.options.namespace + "-pane-container";
    },
    previewImg: function() {
      return "." + this.options.namespace + "-base-container .preview-box";
    },
    scrollerIconFirst: function() {
      if (["top", "bottom"].includes(this.options.scrollerPosition)) {
        if (this.options.scrollerButtonStyle === 'line') {
          return LineLeft;
        } else {
          return FillLeft;
        }
      } else if (["left", "right"].includes(this.options.scrollerPosition)) {
        if (this.options.scrollerButtonStyle === 'line') {
          return LineUp;
        } else {
          return FillUp;
        }
      }
    },
    scrollerIconSecond: function() {
      if (["top", "bottom"].includes(this.options.scrollerPosition)) {
        if (this.options.scrollerButtonStyle === 'line') {
          return LineRight;
        } else {
          return FillRight;
        }
      } else if (["left", "right"].includes(this.options.scrollerPosition)) {
        if (this.options.scrollerButtonStyle === 'line') {
          return LineDown;
        } else {
          return FillDown;
        }
      }
    }
  },
  mounted() {
    if (
      !["left", "right", "top", "bottom"].includes(
        this.options.scrollerPosition
      )
    ) {
      throw "scrollerPosition is invalid";
    }
    if (!["fill", "line"].includes(this.options.scrollerButtonStyle)) {
      throw "scrollerButtonStyle is invalid";
    }
    if (!["left", "right"].includes(this.options.zoomerPanePosition)) {
      throw "zoomerPanePosition is invalid";
    }
    this.$nextTick(() => {
      this[actionName(this.options.scrollerPosition)]();
      this.options.injectBaseStyles = true;
      if (this.options.pane === "container-round") {
        this.options.inlinePane = true;
      } else {
        this.options.inlinePane = false;
        this.options.paneContainer = document.getElementById(
          this.paneContainerId
        );
        let rect = document
          .querySelector("." + this.options.namespace + "-base-container")
          .getBoundingClientRect();
        document
          .getElementById(this.paneContainerId)
          .setAttribute(
            "style",
            getCaculatedPanePosition(
              this.options.pane,
              rect,
              this.options.zoomerPanePosition
            )
          );
      }
      this.drift = new Drift(
        document.querySelector(this.previewImg),
        this.options
      );
    });
  },
  watch: {
    choosedThumb: function(thumb) {
      let matchNormalImg = this.normalSize.find(img => {
        return img.id === thumb.id;
      });
      let matchLargeImg = this.largeSize.find(img => {
        return img.id === thumb.id;
      });
      this.previewLargeImg = Object.assign({}, matchLargeImg);
      this.previewImg = Object.assign({}, matchNormalImg);
      if (this.drift !== null) {
        this.drift.setZoomImageURL(matchLargeImg.url);
      }
    }
  },
  created() {
    if (Object.keys(this.baseImages).length > 0) {
      for (const key in this.baseImages) {
        if (this.baseImages.hasOwnProperty(key)) {
          this[key] = this.baseImages[key];
        }
      }
    }

    if (this.normalSize.length === 0) {
      throw "Product Zoomer Need Normal Size Image At Least!!!";
    }
    if (this.thumbs.length === 0) {
      this.thumbs = Object.assign([], this.normalSize);
    } else {
      this.choosedThumb = this.thumbs[0];
    }
    if (this.largeSize.length === 0) {
      this.largeSize = Object.assign([], this.normalSize);
    }
    if (Object.keys(this.baseZoomerOptions).length > 0) {
      for (const key in this.baseZoomerOptions) {
        if (this.baseZoomerOptions.hasOwnProperty(key)) {
          const element = this.baseZoomerOptions[key];
          this.options[key] = element;
        }
      }
    }

    if (
      this.options.pane === "container-round" ||
      this.options.pane === "container"
    ) {
      this.options.hoverBoundingBox = false;
    } else {
      this.options.hoverBoundingBox = true;
    }
  },
  methods: {
    moveThumbs(direction) {
      let len = this.thumbs.length;
      if (direction === "backward") {
        const moveThumb = this.thumbs.splice(len - 1, 1);
        this.thumbs = [moveThumb[0], ...this.thumbs];
      } else {
        const moveThumb = this.thumbs.splice(0, 1);
        this.thumbs = [...this.thumbs, moveThumb[0]];
      }
    },
    chooseThumb(thumb, event) {
      let eventType = event.type;
      if (eventType === "mouseover") {
        if (this.options.moveByClick !== true) {
          this.choosedThumb = thumb;
        }
      } else {
        this.choosedThumb = thumb;
      }
    },
    scrollerAtBottom() {
      let scrollerItemsCount =
        parseInt(this.baseZoomerOptions.scrollItems) + 2;
      let previewImg = document.querySelector(
        "." + this.options.namespace + "-base-container " + ".preview-box"
      );
      let thumbList = document.querySelector(
        "." + this.options.namespace + "-base-container " + ".thumb-list"
      );
      let thumbListHeight = thumbList.children[1].naturalHeight * (previewImg.naturalWidth / thumbList.children[1].naturalWidth) / (scrollerItemsCount - 1)
      document
        .querySelector("." + this.options.namespace + "-base-container")
        .setAttribute(
          "style",
          "height:" +
            (previewImg.naturalHeight + thumbListHeight + 2) +
            "px;width:" +
            previewImg.naturalHeight +
            "px;position:relative"
        );
      document
        .querySelector(
          "." + this.options.namespace + "-base-container " + ".thumb-list"
        )
        .setAttribute(
          "style",
          "width:" +
            previewImg.naturalWidth +
            "px;height:" +
            thumbListHeight +
            "px;grid-template-columns:calc(100%/" +
            scrollerItemsCount +
            "/2) repeat(" +
            (scrollerItemsCount - 2) +
            ", auto) calc(100%/" +
            scrollerItemsCount +
            "/2);visibility:visible;"
        );
    },
    scrollerAtTop() {
      let scrollerItemsCount =
        parseInt(this.baseZoomerOptions.scrollItems) + 2;
      let previewImg = document.querySelector(
        "." + this.options.namespace + "-base-container " + ".preview-box"
      );
      let thumbList = document.querySelector(
        "." + this.options.namespace + "-base-container " + ".thumb-list"
      );
      let thumbListHeight = thumbList.children[1].naturalHeight * (previewImg.naturalWidth / thumbList.children[1].naturalWidth) / (scrollerItemsCount - 1)
      document
        .querySelector("." + this.options.namespace + "-base-container")
        .setAttribute(
          "style",
          "height:" +
            (previewImg.naturalHeight + thumbListHeight + 2) + // 2px for grid gap
            "px;width:" +
            previewImg.naturalHeight +
            "px;position:relative"
        );
      document
        .querySelector(
          "." + this.options.namespace + "-base-container " + ".thumb-list"
        )
        .setAttribute(
          "style",
          "width:" +
            previewImg.naturalWidth +
            "px;height:" +
            thumbListHeight +
            "px;grid-template-columns:calc(100%/" +
            scrollerItemsCount +
            "/2) repeat(" +
            (scrollerItemsCount - 2) +
            ", auto) calc(100%/" +
            scrollerItemsCount +
            "/2);visibility:visible;"
        );
    },
    scrollerAtRight() {
      let scrollerItemsCount =
        parseInt(this.baseZoomerOptions.scrollItems) + 2;
      let previewImg = document.querySelector(
        "." + this.options.namespace + "-base-container " + ".preview-box"
      );
      let thumbList = document.querySelector(
        "." + this.options.namespace + "-base-container " + ".thumb-list"
      );
      let thumbListWidth = thumbList.children[1].naturalWidth * (previewImg.naturalHeight / thumbList.children[1].naturalHeight) / (scrollerItemsCount - 1)
      document
        .querySelector("." + this.options.namespace + "-base-container")
        .setAttribute(
          "style",
          "width:" +
            (previewImg.naturalWidth + thumbListWidth + 2) +
            "px;position:relative"
        );
      document
        .querySelector(
          "." + this.options.namespace + "-base-container " + ".thumb-list"
        )
        .setAttribute(
          "style",
          "height:" +
            previewImg.naturalHeight +
            "px;width:" +
            thumbListWidth +
            "px;grid-template-rows:calc(100%/" +
            scrollerItemsCount +
            "/2) repeat(" +
            (scrollerItemsCount - 2) +
            ", auto) calc(100%/" +
            scrollerItemsCount +
            "/2);visibility:visible;"
        );
    },
    scrollerAtLeft() {
      let scrollerItemsCount =
        parseInt(this.baseZoomerOptions.scrollItems) + 2;
      let previewImg = document.querySelector(
        "." + this.options.namespace + "-base-container " + ".preview-box"
      );
      let thumbList = document.querySelector(
        "." + this.options.namespace + "-base-container " + ".thumb-list"
      );
      let thumbListWidth = thumbList.children[1].naturalWidth * (previewImg.naturalHeight / thumbList.children[1].naturalHeight) / (scrollerItemsCount - 1)
      document
        .querySelector("." + this.options.namespace + "-base-container")
        .setAttribute(
          "style",
          "width:" +
            (previewImg.naturalWidth + thumbListWidth + 2) + // 2px for grid gap
            "px;position:relative"
        );
      document
        .querySelector(
          "." + this.options.namespace + "-base-container " + ".thumb-list"
        )
        .setAttribute(
          "style",
          "height:" +
            previewImg.naturalHeight +
            "px;width:" +
            thumbListWidth +
            "px;grid-template-rows:calc(100%/" +
            scrollerItemsCount +
            "/2) repeat(" +
            (scrollerItemsCount - 2) +
            ", auto) calc(100%/" +
            scrollerItemsCount +
            "/2);visibility:visible;"
        );
    }
  }
};
</script>
<style>
@import "../assets/drift-zoom/src/css/drift-basic.css";
</style>

<style scoped>
.scroller-at-top {
  display: grid;
  grid-gap: 2px;
  grid-template-columns: 1fr;
  align-items: center;
}

.scroller-at-top .preview-box {
  grid-column: 1 / 2;
  grid-row: 2 / 3;
}

.scroller-at-top .thumb-list {
  display: grid;
  align-items: center;
  grid-column-gap: 0.2em;
  grid-column: 1 / 2;
  grid-row: 1 / 2;
  visibility: hidden;
}
.scroller-at-bottom {
  display: grid;
  grid-gap: 2px;
  grid-template-columns: 1fr;
  align-items: center;
}

.scroller-at-bottom .preview-box {
  grid-column: 1 / 2;
  grid-row: 1 / 2;
}

.scroller-at-bottom .thumb-list {
  display: grid;
  align-items: center;
  grid-column-gap: 0.2em;
  grid-column: 1 / 2;
  grid-row: 2 / 3;
  visibility: hidden;
}

.scroller-at-left {
  display: grid;
  grid-gap: 2px;
  grid-template-columns: 1fr;
}

.scroller-at-left .preview-box {
  grid-column: 2 / 3;
  grid-row: 1 / 2;
}

.scroller-at-left .thumb-list {
  display: grid;
  grid-row-gap: 0.2em;
  grid-column: 1 / 2;
  grid-row: 1 / 2;
  visibility: hidden;
  justify-items: center;
}

.scroller-at-right {
  display: grid;
  grid-gap: 2px;
  grid-template-columns: 1fr;
}

.scroller-at-right .preview-box {
  grid-column: 1 / 2;
  grid-row: 1 / 2;
}

.scroller-at-right .thumb-list {
  display: grid;
  grid-row-gap: 0.2em;
  grid-column: 2 / 3;
  grid-row: 1 / 2;
  visibility: hidden;
  justify-items: center;
}

.scroller-at-right .thumb-list .responsive-image,
.scroller-at-left .thumb-list .responsive-image {
  width: auto;
  height: 100%;
}

.scroller-at-top .thumb-list .responsive-image,
.scroller-at-bottom .thumb-list .responsive-image {
  height: auto;
  width: 100%;
}

.zoomer-control {
  cursor: pointer;
}
.choosed-thumb {
  border-radius: 0px;
}
.pane-container {
  display: none;
  position: absolute;
  z-index: 10000;
  pointer-events: none;
}
</style>
