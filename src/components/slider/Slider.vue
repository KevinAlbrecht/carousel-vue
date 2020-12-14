<template>
  <div class="slider_main">
    <div class="header">
      <h2 class="title">{{ title }}</h2>
      <ul class="pagination-indicator">
        <li
          v-for="page in pagesCount"
          :key="page"
          :class="{ active: page === activePage }"
        ></li>
      </ul>
    </div>
    <div class="slider_container" :style="computeSliderCssVariables">
      <span
        @click="slide(true)"
        class="arrow arrow-prev"
        :class="{ disabled: computeLeftNavDisabled }"
        ><b></b
      ></span>
      <div
        v-if="vItems"
        :class="['slider_slide', inTransition ? 'in-transition' : '']"
        :style="{
          transform: sliderTransform,
        }"
      >
        <div
          class="box"
          :style="computeBoxWidth"
          v-for="item in vItems"
          :key="item.id"
        >
          <slot :item="item"></slot>
        </div>
      </div>
      <span
        @click="slide()"
        class="arrow arrow-next"
        :class="{ disabled: computeRightNavDisabled }"
        ><b></b
      ></span>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    items: Array,
    title: String,
    chunkSize: {
      type: Number,
      default: 6,
    },
    translationDuration: {
      type: Number,
      default: 700,
    },
    loop: {
      type: Boolean,
      default: true,
    },
  },
  data() {
    return {
      $_isInfinite: false,
      pagesCount: 0,
      activePage: 1,
      vItems: [],
      inTransition: false,
      translate: 0,
      rawBoxPart: 20,
    };
  },
  methods: {
    slide(isLeft = false) {
      console.log("vier");
      if (
        this.inTransition === true ||
        (!this.loop &&
          ((!isLeft && this.activePage === this.pagesCount) ||
            (isLeft && this.activePage === 1)))
      )
        return;

      this.$emit(Events.slide, {
        currentChunk: this.activePage,
        directionLeft: 0,
      });

      if (this.loop === false) {
        this.activePage = computeActivePage(
          this.pagesCount,
          this.activePage,
          isLeft
        );

        setTimeout(() => {
          this.inTransition = true;
          isLeft
            ? (this.translate += this.translationUnity)
            : (this.translate -= this.translationUnity);
        }, 0);
        setTimeout(() => {
          this.inTransition = false;
        }, this.translationDuration);
      } else {
        this.activePage = computeActivePage(
          this.pagesCount,
          this.activePage,
          isLeft
        );

        if (this.$_isInfinite) {
          preReorder(this.vItems, isLeft);
          this.translate += isLeft
            ? -this.computeItemRatio
            : this.computeItemRatio;
        }
        setTimeout(() => {
          this.inTransition = true;
          isLeft
            ? (this.translate += this.translationUnity)
            : (this.translate -= this.translationUnity);
        }, 0);
        setTimeout(() => {
          this.inTransition = false;
          if (this.$_isInfinite) {
            this.translate = -this.translationUnity;
            reorder(this.vItems, this.chunkSize, isLeft);
          } else this.$_isInfinite = true;
        }, this.translationDuration);
      }
    },
  },
  computed: {
    translationUnity() {
      return translationMovmentUnity;
    },
    computeItemRatio() {
      return 100 / this.chunkSize;
    },
    sliderTransform() {
      return `translateX(calc(${this.translate}% + ${Math.abs(
        this.translate
      )}px))`;
    },
    computeBoxWidth() {
      return `width: calc(${this.computeItemRatio}% - 10px + 5px);`;
    },
    computeSliderCssVariables() {
      return {
        "--slider-transition-speed": this.translationDuration + "ms",
      };
    },
    computeLeftNavDisabled() {
      return computeIsNavDisabled(
        this.loop,
        this.pagesCount,
        this.activePage,
        true
      );
    },
    computeRightNavDisabled() {
      return computeIsNavDisabled(this.loop, this.pagesCount, this.activePage);
    },
  },
  watch: {
    items: function (newVal) {
      this.vItems = newVal;
      this.pagesCount = Math.floor(this.vItems.length / this.chunkSize);
    },
  },
};

const Events = { slide: "slide" };

const translationMovmentUnity = 100;

const computeIsNavDisabled = (loop, pagesCount, activePage, isLeft = false) => {
  return (
    pagesCount > 0 &&
    !loop &&
    ((isLeft && activePage > 1) || (!isLeft && activePage < pagesCount - 1))
  );
};

const computeActivePage = (pagesCount, activePage, isLeft = false) => {
  if (activePage === pagesCount && !isLeft) activePage = 1;
  else if (activePage === 1 && isLeft) activePage = pagesCount;
  else activePage += isLeft ? -1 : +1;

  return activePage;
};

const preReorder = (vItems, isLeft = false) => {
  if (isLeft) vItems.unshift(...vItems.splice(vItems.length - 1, 1));
  if (!isLeft) vItems.push(...vItems.splice(0, 1));
};

const reorder = (vItems, chunkSize, isLeft = false) => {
  const itemsCount = vItems.length;
  const itemsToReorder = chunkSize;
  if (isLeft)
    vItems.unshift(
      ...vItems.splice(itemsCount - itemsToReorder + 1, itemsToReorder)
    );
  else vItems.push(...vItems.splice(0, itemsToReorder - 1));
};
</script>
<style scoped>
/* HEADER */
.slider_main {
  overflow: hidden;
}
.slider_main .header {
  padding: 0 10px 0 50px;
  display: flex;
  align-items: center;
}
.slider_main .header h2.title {
  color: white;
  flex: 1;
}

/* HEADER pagination */
.slider_main .header ul.pagination-indicator {
  list-style-type: none;
  display: none;
}
.slider_main:hover .header ul.pagination-indicator {
  display: inline;
}
.slider_main .header ul.pagination-indicator li {
  display: inline-block;
  width: 12px;
  height: 2px;
  background-color: #4d4d4d;
  margin-left: 1px;
}

.slider_main .header ul.pagination-indicator li.active {
  background-color: #aaa;
}

/* CONTAINER */
.slider_main .slider_container {
  position: relative;
}

/* ARROW */

.slider_main .slider_container .arrow {
  position: absolute;
  z-index: 1;
  top: 0;
  bottom: 0;
  width: 50px;
  background-color: rgba(20, 20, 20, 0.5);
  display: flex;
  justify-content: center;
  cursor: pointer;
}
.slider_main .slider_container .arrow b {
  display: block;
  align-self: center;
  height: 20px;
  line-height: 20px;
  position: relative;
}

.slider_main .slider_container .arrow.arrow-prev {
  left: 0;
}

.slider_main .slider_container .arrow.arrow-next {
  right: 0;
}

.slider_main .slider_container .arrow b::before,
.slider_main .slider_container .arrow b::after {
  color: white;
  font-weight: 700;
  font-size: 25px;
}

.slider_main .slider_container .arrow.arrow-prev b::before {
  content: "";
  display: block;
  border: solid #fff;
  border-width: 0 0 3px 3px;
  position: absolute;
  top: 4px;
  width: 14px;
  height: 14px;
  transform: translateX(-50%) rotate(45deg);
}

.slider_main .slider_container .arrow.arrow-next b::after {
  content: "";
  display: block;
  border: solid #fff;
  border-width: 3px 3px 0 0;
  position: absolute;
  top: 4px;
  width: 14px;
  height: 14px;
  transform: translateX(-50%) rotate(45deg);
}

/* Slide */

.slider_main .slider_container .slider_slide {
  overflow-x: visible;
  padding: 0 50px;
  white-space: nowrap;
  position: relative;
  line-height: 0;
}

.slider_main .slider_container .slider_slide.in-transition {
  transition: transform var(--slider-transition-speed);
}

.slider_main .slider_container .slider_slide .box {
  display: inline-block;
  margin-right: 5px;
}

.slider_main .slider_container .slider_slide .box .img_container {
  width: 100%;
  height: 0;
  position: relative;
  overflow: hidden;
  padding-top: 150%;
}

.slider_main .slider_container .slider_slide .box .img_container img {
  position: absolute;
  max-width: 100%;
  max-height: 100%;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
}
</style>