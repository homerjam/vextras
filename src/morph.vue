<script>
/**
 * vuxtras/morph
 *
 * Morph component
 *
 * Usage:
 *
 * <morph :watch="booleanProp">
 *
 *  <svg>
 *    <!-- default state -->
 *  </svg>
 *
 *  <svg>
 *    <!-- active state -->
 *  </svg>
 *
 * </morph>
 *
 */

import { Tween, Easing, autoPlay } from 'es6-tween';
import { interpolate } from 'polymorph-js';
import { reverse } from 'svg-path-reverse';

export default {
  props: {
    watch: {
      type: Boolean,
      required: true,
    },
    speed: {
      type: Number,
      default: 200,
    },
    reverse: {
      type: Boolean,
      default: false,
    },
    viewBox: {
      type: String,
      default: undefined,
    },
    preserveAspectRatio: {
      type: String,
      default: undefined,
    },
    width: {
      type: [Number, String],
      default: undefined,
    },
    height: {
      type: [Number, String],
      default: undefined,
    },
  },
  data() {
    return {
      ready: false,
      targetViewBox: undefined,
      targetPreserveAspectRatio: undefined,
      targetPath: null,
      sourcePaths: null,
      renderMorph: null,
      offset: this.watch ? 1 : 0,
    };
  },
  watch: {
    watch() {
      new Tween(this)
        .to({ offset: this.watch ? 1 : 0 }, this.speed)
        .easing({
          offset: Easing.Quadratic.InOut,
        })
        .on('update', this.render)
        .start();
    },
  },
  mounted() {
    autoPlay(true);

    const sourceSvgs = this.$el.querySelectorAll('.morph-sources svg');

    const width = this.width || sourceSvgs[0].clientWidth;
    const height = this.height || sourceSvgs[0].clientHeight;

    this.$el.style.width = typeof width === 'string' ? width : `${width}px`;
    this.$el.style.height = typeof height === 'string' ? height : `${height}px`;

    this.targetViewBox =
      this.viewBox ||
      sourceSvgs[0].getAttribute('viewBox') ||
      sourceSvgs[0].getAttribute('viewbox') ||
      '0 0 24 24';

    this.targetPreserveAspectRatio =
      this.preserveAspectRatio ||
      sourceSvgs[0].getAttribute('preserveAspectRatio') ||
      sourceSvgs[0].getAttribute('preserveAspectRatio') ||
      'xMidYMid';

    this.targetPath = this.$el.querySelector('.morph-target-path');

    this.sourcePaths = [
      ...this.$el.querySelectorAll('.morph-sources svg path'),
    ];

    if (this.reverse) {
      this.sourcePaths[1].setAttribute(
        'd',
        reverse(this.sourcePaths[1].getAttribute('d'))
      );
    }

    this.renderMorph = interpolate(this.sourcePaths, {
      optimize:
        Math.round(this.sourcePaths[0].getTotalLength()) ===
        Math.round(this.sourcePaths[1].getTotalLength())
          ? 'none'
          : 'fill',
    });

    this.ready = true;

    this.render();
  },
  methods: {
    render() {
      this.targetPath.setAttribute('d', this.renderMorph(this.offset));
    },
  },
};
</script>

<template>
  <div class="morph" :class="{ ready }">
    <span class="morph-sources">
      <slot />
    </span>
    <svg
      :viewBox="targetViewBox"
      :preserveAspectRatio="targetPreserveAspectRatio"
      class="morph-target"
    >
      <path class="morph-target-path" />
    </svg>
  </div>
</template>

<style>
.morph {
  position: relative;
  display: inline-block;
  vertical-align: middle;
  width: 24px;
  height: 24px;
}

.morph-sources svg {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  user-select: none;
  pointer-events: none;
}

.morph-sources svg:not(:first-child) {
  visibility: hidden;
}

.morph.ready .morph-sources svg:first-child {
  visibility: hidden;
}

.morph-target {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>
