<template>
  <div
    class="drag-container"
    ref="container"
    @mousedown="mousedown"
    @mouseup="mouseup"
    @mouseleave="mouseleave"
    @mousemove="mousemove"
  >
    <span
      class="char"
      :ref="'ch' + index"
      v-for="(ch, index) in question"
      :key="index + ch"
      >{{ ch }}</span
    >
  </div>
</template>

<script>
export default {
  props: ["question"],
  created() {
    console.log(this.$refs);
  },
  data() {
    const div = document.createElement("div");
    div.classList.add("__CG_rect");
    return {
      selected: [],
      div,
      active: false,

      startPosition: { x: 0, y: 0 },
      threshold: 60,
      latch: false,
    };
  },
  computed: {
    chars() {
      return Object.keys(this.$refs)
        .filter(it => it.startsWith("ch"))
        .reduce((p, c) => {
          const dom = this.$refs[c][0];
          const cr = dom.getBoundingClientRect();
          const pcr = this.$refs["container"].getBoundingClientRect();
          return [
            ...p,
            {
              dom: this.$refs[c][0],
              rect: {
                top: cr.top - pcr.top,
                left: cr.left - pcr.left,
                bottom: cr.bottom - pcr.top,
                right: cr.right - pcr.left,
              },
            },
          ];
        }, []);
    },
  },
  methods: {
    reset() {
      this.active = false;
      this.selected = [];
      this.chars.forEach(it => (it.dom.style.backgroundColor = "white"));
    },

    mousedown(e) {
      if (!this.active) {
        console.log("mousedown", e);
        this.startPosition = { x: e.offsetX, y: e.offsetY };
        this.div.style.left = `${this.startPosition.x}px`;
        this.div.style.top = `${this.startPosition.y}px`;
        this.div.style.width = `${0}px`;
        this.div.style.height = `${0}px`;
        this.$refs.container.appendChild(this.div);
        this.reset();
        this.active = true;
      }
    },
    mousemove(e) {
      if (this.active) {
        console.log("mousemove");
        const x = e.offsetX;
        const y = e.offsetY;

        let left = 0,
          top = 0;

        const width = Math.abs(x - this.startPosition.x),
          height = Math.abs(y - this.startPosition.y);

        if (x - this.startPosition.x >= 0) {
          left = this.startPosition.x;
        } else {
          left = this.startPosition.x - Math.abs(x - this.startPosition.x);
        }

        if (y - this.startPosition.y >= 0) {
          top = this.startPosition.y;
        } else {
          top = this.startPosition.y - Math.abs(y - this.startPosition.y);
        }

        this.div.style.left = `${left}px`;
        this.div.style.top = `${top}px`;
        this.div.style.width = `${width}px`;
        this.div.style.height = `${height}px`;

        if (!this.latch) {
          this.latch = true;
          setTimeout(() => {
            this.latch = false;
          }, this.threshold);
          this.checkInside(left, top, width, height);
        }
      }
    },

    checkInside(x, y, w, h) {
      console.log("check inside", x, y, w, h);
      this.selected = [];
      for (const char of this.chars) {
        const rect = char.rect;
        const ch = char.dom;
        console.log("----------------------------");
        console.log("ch", ch);
        console.log(`${rect.left} < ${x} + ${w} && ${rect.top} < ${y + h}`);
        console.log("----------------------------");
        if (
          // left && top
          (rect.left < x + w &&
            rect.top < y + h &&
            rect.left > x &&
            rect.top > y) ||
          (rect.left < x + w &&
            rect.bottom > y &&
            rect.left > x &&
            rect.bottom < y + h) ||
          (rect.right > x &&
            rect.top < y + h &&
            rect.right < x + w &&
            rect.top > y) ||
          (rect.right > x &&
            rect.bottom > y &&
            rect.right < x + w &&
            rect.bottom < y + h)
        ) {
          this.selected.push(ch);
          ch.style.backgroundColor = "red";
        } else {
          ch.style.backgroundColor = "white";
        }
      }
    },

    fireEvent() {
      this.$emit("selectedChanged", [...this.selected]);
    },

    // eslint-disable-next-line no-unused-vars
    mouseup(e) {
      if (this.active) {
        console.log("mouseup");
        this.$refs.container.removeChild(this.div);
        this.fireEvent();
        this.reset();
      }
    },
    // eslint-disable-next-line no-unused-vars
    mouseleave(e) {
      if (this.active) {
        console.log("mouseleave");
        this.$refs.container.removeChild(this.div);
        this.fireEvent();
        this.reset();
      }
    },
  },
};
</script>

<style>
.drag-container {
  padding: 5rem;
  background: #ccc;
  position: relative;
}

.char {
  font-size: 1.5rem;
  user-select: none;
}

.__CG_rect {
  border: red solid 1px;
  position: absolute;
  width: 50px;
  height: 50px;
  pointer-events: none;
}
</style>
