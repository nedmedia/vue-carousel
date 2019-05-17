<template>
  <div
    class="VueCarousel-slide"
    tabindex="-1"
    :aria-hidden="!isActive"
    role="tabpanel"
    v-bind:style="`
      -ms-flex-preferred-size: ${slideWidth > 1 ? slideWidth + 'px' : 'inherit'};
      flex-basis: ${slideWidth > 1 ? slideWidth + 'px' : 'inherit'};
      ${slideWidth > 1 ? 'width:' + slideWidth + 'px' : ''};
    `"
    :class="{
      'VueCarousel-slide-active': isActive,
      'VueCarousel-slide-center': isCenter,
      'VueCarousel-slide-adjustableHeight': isAdjustableHeight
    }"
  >
    <slot></slot>
  </div>
</template>

<script>
export default {
  name: "slide",
  props: ["title"],
  data() {
    return {
      width: null
    };
  },
  inject: ["carousel"],
  mounted() {
    if (!this.$isServer) {
      this.$el.addEventListener("dragstart", e => e.preventDefault());
    }

    this.$el.addEventListener(
      this.carousel.isTouch ? "touchend" : "mouseup",
      this.onTouchEnd
    );
  },
  computed: {
    activeSlides() {
      const { currentPage, breakpointSlidesPerPage, $children } = this.carousel;
      const activeSlides = [];
      const children = $children
        .filter(
          child =>
            child.$el && child.$el.className.indexOf("VueCarousel-slide") >= 0
        )
        .map(child => child._uid);

      let i = 0;
      while (i < breakpointSlidesPerPage) {
        const child = children[currentPage * breakpointSlidesPerPage + i];
        activeSlides.push(child);
        i++;
      }

      return activeSlides;
    },
    /**
     * `isActive` describes whether a slide is visible
     * @return {Boolean}
     */
    isActive() {
      return this.activeSlides.indexOf(this._uid) >= 0;
    },
    /**
     * `isCenter` describes whether a slide is in the center of all visible slides
     * if perPage is an even number, we quit
     * @return {Boolean}
     */
    isCenter() {
      const { breakpointSlidesPerPage } = this.carousel;
      if (breakpointSlidesPerPage % 2 === 0 || !this.isActive) return false;
      return (
        this.activeSlides.indexOf(this._uid) ===
        Math.floor(breakpointSlidesPerPage / 2)
      );
    },
    /**
     * `isAdjustableHeight` describes if the carousel adjusts its height to the active slide(s)
     * @return {Boolean}
     */
    isAdjustableHeight() {
      const { adjustableHeight } = this.carousel;
      return adjustableHeight;
    },
    slideWidth() {
      return this.carousel.slideWidth;
    }
  },
  methods: {
    onTouchEnd(e) {
      /**
       * @event slideclick
       * @event slide-click
       * @type {Object}
       */
      const eventPosX =
        this.carousel.isTouch && e.changedTouches && e.changedTouches.length > 0
          ? e.changedTouches[0].clientX
          : e.clientX;
      const deltaX = this.carousel.dragStartX - eventPosX;

      if (
        this.carousel.minSwipeDistance === 0 ||
        Math.abs(deltaX) < this.carousel.minSwipeDistance
      ) {
        this.$emit("slideclick", Object.assign({}, e.currentTarget.dataset));
        this.$emit("slide-click", Object.assign({}, e.currentTarget.dataset));
      }
    }
  }
};
</script>

<style>
.VueCarousel-slide {
  -ms-flex-preferred-size: inherit;
  flex-basis: inherit;
  -webkit-box-flex: 0;
  -ms-flex-positive: 0;
  flex-grow: 0;
  -ms-flex-negative: 0;
  flex-shrink: 0;
  flex-basis: inherit;
  flex-grow: 0;
  user-select: none;
  backface-visibility: hidden;
  -webkit-touch-callout: none;
  -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
  outline: none;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

.VueCarousel-slide-adjustableHeight {
  display: table;
  flex-basis: auto;
  width: 100%;
}
</style>
