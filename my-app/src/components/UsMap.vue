<template>
    <div class="map-container" ref="containerRef">
      <svg
        v-if="features.length"
        :width="svgWidth"
        :height="svgHeight"
        tabindex="0"
        aria-label="US States Map"
      >
        <g>
          <template v-for="feature in features" :key="feature.properties.STATE">
            <path
              class="state-polygon"
              :d="getPath(feature)"
              :fill="getFill(feature)"
              :tabindex="0"
              :aria-label="feature.properties.NAME"
              @mouseenter="hovered = feature.properties.STATE"
              @mouseleave="hovered = null"
              @focus="hovered = feature.properties.STATE"
              @blur="hovered = null"
              @click="emitClick(feature)"
            />
          </template>
        </g>
      </svg>
      <div v-else style="text-align:center; padding:2em;">Loading map…</div>
    </div>
  </template>
  
  <script setup>
  import { ref, reactive, onMounted, watch } from 'vue'
  import * as d3 from 'd3'
  
  const props = defineProps({
    budgets: {
      type: Object,
      required: true,
    },
  })
  const emit = defineEmits(['state-click'])
  
  const geojsonUrl =
    'https://eric.clst.org/assets/wiki/uploads/Stuff/gz_2010_us_040_00_500k.json'
  const features = ref([])
  const svgWidth = 980
  const svgHeight = 600
  const projection = d3.geoAlbersUsa().fitSize([svgWidth, svgHeight], {type: 'FeatureCollection', features: []})
  const pathGen = d3.geoPath().projection(projection)
  const hovered = ref(null)
  
  // Fit projection after data loaded
  function fitProjection() {
    if (features.value.length) {
      projection.fitSize([svgWidth, svgHeight], { type: 'FeatureCollection', features: features.value })
    }
  }
  
  onMounted(async () => {
    const resp = await fetch(geojsonUrl)
    const geojson = await resp.json()
    features.value = geojson.features
    fitProjection()
  })
  
  watch(features, fitProjection)
  
  function getPath(feature) {
    return pathGen(feature)
  }
  
  // Color interpolation
  function getFill(feature) {
    const fips = feature.properties.STATE
    const val = props.budgets[fips]
    if (val == null) return '#eeeeee'
    // Linear green scale (0=light, 1000=dark)
    const t = Math.max(0, Math.min(val, 1000)) / 1000
    return d3.interpolateGreens(0.2 + t * 0.7)
  }
  
  function emitClick(feature) {
    emit('state-click', {
      name: feature.properties.NAME,
      fips: feature.properties.STATE,
    })
  }
  </script>
  
  <style scoped>
  .map-container {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100vw;
    max-width: 100vw;
    overflow-x: auto;
    margin: 2rem 0 0 0;
  }
  svg {
    background: white;
    border-radius: 14px;
    box-shadow: 0 4px 24px rgba(30,40,50,0.10);
    width: 98vw;
    height: auto;
    max-width: 980px;
  }
  </style>
  