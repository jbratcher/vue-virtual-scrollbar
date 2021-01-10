<template>
  <!-- Scrollbar -->
  <div id="content-scrollbar">
    <div id="content-scrollbar-thumb"></div>
  </div>
</template>

<script>
export default {
  name: "VirtualScrollbar",
  props: {
    containerId: {
      type: String,
      default: "content",
    },
  },
  data() {
    return {
      contentContainer: document.getElementById(this.containerId),
      containerContainerHeight: null,
      contentContainerId: this.containerId,
      scrollbar: null,
      scrollPosition: null,
      scrollPositionMax: null,
      scrollPositionValue: null,
      scrollbarThumb: null,
      scrollbarThumbHeight: null,
      viewportRatio: null,
    };
  },
  methods: {
    // create a custom scrollbar element that tracks the scroll position of the container from the top of the page
    createMirroredScrollbar(elementId) {
      // content container element (scroll container)
      this.contentContainer = document.getElementById(elementId);
      this.scrollbar = document.getElementById("content-scrollbar");
      this.scrollbarThumb = document.getElementById("content-scrollbar-thumb");

      // set initial height of scrollbar thumb to mirror browser scrollbar height
      this.viewportRatio =
        window.innerHeight /
        this.contentContainer.getBoundingClientRect().height;
      if (this.viewportRatio > 1) {
        this.viewportRatio = 1;
      }

      // thumbnail height as percentage of scroll container
      this.scrollbarThumbHeight = Math.max(
        1,
        Math.floor(
          this.scrollbar.getBoundingClientRect().height * this.viewportRatio
        )
      );
      this.scrollbarThumb.style.height = `${this.scrollbarThumbHeight / 16}rem`;
      // set max scroll position for thumb bottom edge case
      this.scrollPositionMax = Math.ceil((1 - this.viewportRatio) * 100);

      // when user scrolls in content container
      window.addEventListener("scroll", this.handleContainerScroll);
      window.addEventListener("resize", () => {
        window.addEventListener("scroll", this.handleContainerScroll);
      });
    },
    // calculates the scroll thumb position from container scroll position
    handleContainerScroll() {
      console.log("scrolling");

      // set position as percentage of distance (top / height)
      this.scrollPosition = String(
        (
          Math.abs(
            this.contentContainer.getBoundingClientRect().y /
              this.contentContainer.getBoundingClientRect().height
          ) * 100
        ).toFixed(0) + "%"
      );

      // format current scroll position as percentage for style
      this.scrollPositionValue = Number(this.scrollPosition.replace("%", ""));
      console.log("Scroll position value: ", this.scrollPositionValue);
      // set scroll thumb position with top and bottom edge cases
      // if the scroll position is maxed out, pin to bottom
      if (this.scrollPositionValue >= this.scrollPositionMax) {
        this.scrollbarThumb.style.top = `${this.scrollPositionMax}%`;
        this.scrollbarThumb.style.bottom = 0;
      } else if (this.scrollPositionValue <= 5) {
        // pin to top edge case
        this.scrollbarThumb.style.top = 0;
      } else {
        // set thumb position based on scroll position
        this.scrollbarThumb.style.top = `${Number(
          this.scrollPosition.replace("%", "")
        )}%`;
      }
    },
  },
  mounted() {
    this.contentContainer = document.getElementById(this.contentContainerId);
    if (this.contentContainer) {
      this.createMirroredScrollbar(this.contentContainerId);
      // waiting for all components to be loaded and yield correct height
      setTimeout(() => {
        this.$nextTick(() => {
          this.containerHeight = document
            .getElementById("content")
            .getBoundingClientRect().height;
          console.log("Calculated Height: ", this.containerHeight);
          this.contentContainer.style.height = this.containerHeight;
          this.createMirroredScrollbar(this.contentContainerId);
        });
      }, 1000);
    }
  },
};
</script>

<style lang="scss">
// scrollbar
#content-scrollbar {
  background-color: transparent;
  border: 0.125rem solid teal;
  height: 17.5rem;
  width: 1rem;
  position: fixed;
  top: 50%;
  left: 3vw;
  transform: translateY(-50%);
  z-index: 2;

  #content-scrollbar-thumb {
    background: teal;
    height: 2rem;
    max-height: 100%;
    width: 100%;
    position: absolute;
    top: 0;
    transition: all 0.1s linear;
  }
}

#content-scrollbar {
  @media screen and (max-width: 1023px) {
    visibility: hidden;
  }
}
</style>
