<template>
  <div>
    <div id="map" />
    <div id="tooltip" />
  </div>
</template>

<script>
import { extent } from 'd3-array'
import { interpolateOranges, interpolateYlOrRd } from 'd3-scale-chromatic'
import { scaleSequential } from 'd3-scale'
import { color } from 'd3-color'
import mapboxgl from 'mapbox-gl'
import { ScatterplotLayer, TextLayer, GridLayer } from '@deck.gl/layers'
import { MapboxLayer } from '@deck.gl/mapbox'

export default {
  props: {
    data: {
      type: Array,
      default: () => []
    },
    show: {
      type: Boolean,
      default: () => true
    }
  },

  data() {
    return {
      id: 'scatter',
      map: null,
      deckLayer: null,
      layerId: null,
      $tooltip: null
    }
  },

  watch: {
    data(value) {
      if (this.show) {
        if (!this.map.getLayer(this.id)) {
          this.setup()

          this.map.on('load', () => {
            this.map.addLayer(this.deckLayer)
          })
        }
      }
    },
    show(value) {
      if (value) {
        if (!this.map.getLayer(this.id)) {
          this.map.addLayer(this.deckLayer)
        }
      } else {
        this.map.removeLayer(this.id)
      }
    }
  },

  mounted() {
    this.$tooltip = this.$el.querySelector('#tooltip')
    this.setup()
  },

  methods: {
    setTooltip(object, x, y) {
      const el = this.$tooltip
      if (object) {
        el.innerHTML = object.message
        el.style.display = 'block'
        el.style.left = `${x}px`
        el.style.top = `${y}px`
      } else {
        el.style.display = 'none'
      }
    },

    setup() {
      mapboxgl.accessToken =
        'pk.eyJ1Ijoic3RldmVudGFuIiwiYSI6ImNqcW9tNm83ZTd1bXYzeHVsY3c5N3QzMGYifQ.CnFVRNP-R6lnPENtG0a7Hg'
      this.map = new mapboxgl.Map({
        container: 'map',
        style: 'mapbox://styles/mapbox/dark-v8',
        center: [134.489563, -25.734968],
        zoom: 4,
        minZoom: 3,
        maxZoom: 8
        // pitch: 20
      })

      const colourScale = scaleSequential(interpolateYlOrRd).domain(
        extent(this.data, d => d.properties.intensity)
      )

      const data = this.data.map(d => {
        const coords = d.geometry.coordinates
        const intensity = d.properties.intensity
        // if (d.TC_exposed_capital > 0 || d.TC_exposed_pop > 0) {
        //   console.log(d)
        // }
        const colour = color(colourScale(intensity))
        return {
          position: [coords[0], coords[1]],
          color: [colour.r, colour.g, colour.b],
          intensity: intensity,
          radius: intensity * 40
        }
      })

      this.deckLayer = new MapboxLayer({
        id: this.id,
        type: ScatterplotLayer,
        data,
        getPosition: d => d.position,
        getRadius: d => d.radius,
        getColor: d => d.color,
        opacity: 0.1,
        pickable: true,
        onHover: ({ object, x, y }) => {
          const intensity = object ? object.intensity : ''
          const tooltip = `Intensity: ${intensity}`
          if (object) object.message = tooltip
          this.setTooltip(object, x, y)
        }
      })
    }
  }
}
</script>

<style>
#map {
  width: 100%;
  height: 100vh;
}
#tooltip {
  position: absolute;
  color: #fff;
  z-index: 1;
  pointer-events: none;
  background: #111;
  padding: 6px 12px;
  border-radius: 2px;
  box-shadow: 1px 1px 2px rgba(0, 0, 0, 0.2);
}
</style>
