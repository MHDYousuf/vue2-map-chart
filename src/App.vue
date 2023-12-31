<template>
  <div class="vue-world-map">
    <Map
      @hoverCountry="onHoverCountry"
      @hoverLeaveCountry="onHoverLeaveCountry"
      :key="computedKey"
    />

    <div
      v-if="legend.name"
      class="vue-map-legend"
      :style="'left:' + position.left + 'px; top: ' + position.top + 'px'"
    >
      <div class="vue-map-legend-header">
        <slot name="legend_header">
          <span>{{ legend.name }}</span>
        </slot>
      </div>
      <div class="vue-map-legend-content">
        <slot name="legend_content">
          <span v-if="countryData[legend.code]"
            >{{ countryData[legend.code].count }}
            {{ countryData[legend.code].percentage }}</span
          >
          <span v-else>0</span>
        </slot>
      </div>
    </div>
  </div>
</template>

<script>
import chroma from "chroma-js";
import Map from "./Map";
import {
  getDynamicMapCss,
  getBaseCss,
  getCombinedCssString,
} from "./dynamic-map-css";

let legend = {
  data: null,
  code: "",
  name: null,
  percentage: null,
};

let position = {
  left: 0,
  top: 0,
};

export default {
  name: "MapChart",
  components: { Map },

  computed: {
    computedKey() {
      return this.key + this.keyChange;
    },
  },
  props: {
    lowColor: {
      type: String,
      default: "#fde2e2",
    },
    highColor: {
      type: String,
      default: "#d83737",
    },
    countryData: {
      type: Object,
      required: true,
      default() {
        return {
          AE: {
            count: 0,
            percentage: "",
          },
        };
      },
    },
    defaultCountryFillColor: {
      type: String,
      default: "#dadada",
    },
    countryStrokeColor: {
      type: String,
      default: "#909090",
    },
    key: {
      default: 0,
    },
  },
  watch: {
    countryData: {
      deep: true,
      immediate: true,
      handler(newVal,oldVal) {
        this.renderMapCSS();
        // console.log('countryData', newVal)
      },
    },
  },
  data() {
    return {
      legend: legend,
      position: position,
      node: document.createElement("style"),
      chromaScale: chroma.scale([this.lowColor, this.highColor]),
      keyChange: 0,
    };
  },
  methods: {
    onHoverCountry(country) {
      this.legend = country;
      this.position = country.position;
      this.$emit("hoverCountry", country);
    },
    onHoverLeaveCountry(country) {
      this.legend = {
        data: null,
        code: null,
        name: null,
        percentage: null,
      };
      this.$emit("hoverLeaveCountry", country);
    },
    renderMapCSS() {
      document.body.appendChild(this.node);
      const baseCss = getBaseCss(this.$props);
      const dynamicMapCss = getDynamicMapCss(
        this.countryData,
        this.chromaScale
      );
      this.node.innerHTML = getCombinedCssString(baseCss, dynamicMapCss);
      this.keyChange = this.keyChange + 1;
    },
  },
  mounted() {
    document.body.appendChild(this.node);
    this.renderMapCSS();
  },
  destroyed() {
    this.renderMapCSS()
  },
};
</script>

<style scoped>
.vue-world-map,
#map-svg {
  height: 100%;
}

.vue-world-map {
  position: relative;
}

.vue-map-legend {
  width: 185px;
  background: #fff;
  border: 1px solid;
  border-color: #acacad;
  position: absolute;
}

.vue-map-legend-header {
  padding: 10px 15px;
}

.vue-map-legend-content {
  padding: 10px 15px;
  background: #dadbda8f;
  border-top: 1px solid #acacad;
}
</style>
