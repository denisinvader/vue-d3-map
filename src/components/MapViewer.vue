<script>
const d3 = {
  ...require('d3-geo'),
  ...require('d3-tile'),
};

const MIN_ZOOM = 10;
const MAX_ZOOM = 27;

export default {
  props: {
    center: {
      type: Array,
      default: () => [-7.584838, 33.561041],
    },
    initialZoom: {
      type: [Number, String],
      default: 20,
    },
  },
  data () {
    return {
      width: 0,
      height: 0,

      translateX: 0,
      translateY: 0,

      touchStarted: false,
      touchLastX: 0,
      touchLastY: 0,

      zoom: +this.initialZoom,
      scale: 1 << +this.initialZoom,
    };
  },
  computed: {
    projection () {
      return d3.geoMercator()
        .scale(+this.scale / (2 * Math.PI))
        .translate([this.translateX, this.translateY])
        .center(this.center)
      ;
    },
    tiles () {
      return d3.tile()
        .size([this.width, this.height])
        .scale(+this.scale)
        .translate(this.projection([0, 0]))()
      ;
    },
  },
  watch: {
    zoom (zoom, prevZoom) {
      const k = zoom - prevZoom > 0 ? 2 : .5;

      this.scale = 1 << zoom;
      this.translateY = this.height / 2 - k * (this.height / 2 - this.translateY);
      this.translateX = this.width / 2 - k * (this.width / 2 - this.translateX);
    },
  },
  mounted () {
    const rect = this.$el.getBoundingClientRect();

    this.width = rect.width;
    this.height = rect.height;
    this.translateX = this.width / 2;
    this.translateY = this.height / 2;
  },
  methods: {
    onTouchStart (e) {
      this.touchStarted = true;

      this.touchLastX = e.clientX;
      this.touchLastY = e.clientY;
    },
    onTouchEnd () {
      this.touchStarted = false;
    },
    onTouchMove (e) {
      if (this.touchStarted) {
        this.translateX = this.translateX + e.clientX - this.touchLastX;
        this.translateY = this.translateY + e.clientY - this.touchLastY;

        this.touchLastX = e.clientX;
        this.touchLastY = e.clientY;
      }
    },

    zoomIn () {
      this.zoom = Math.min(this.zoom + 1, MAX_ZOOM);
    },
    zoomOut () {
      this.zoom = Math.max(this.zoom - 1, MIN_ZOOM);
    },
  },
  render () {
    if (this.width <= 0 || this.height <= 0) {
      return <div class="map" />;
    }

    return (
      <div class="map">
        <div class="map__controls">
          <button
            class="map__button"
            disabled={this.zoom >= MAX_ZOOM}
            onClick={this.zoomIn}
          >+</button>
          <button
            class="map__button"
            disabled={this.zoom <= MIN_ZOOM}
            onClick={this.zoomOut}
          >-</button>
        </div>

        <svg
          viewBox={`0 0 ${this.width} ${this.height}`}
          onMousedown={this.onTouchStart}
          onMousemove={this.onTouchMove}
          onMouseup={this.onTouchEnd}
          onMouseleave={this.onTouchEnd}
        >
          <g>
            {this.tiles.map((t, i) => (
              <image
                class="map__tile"
                key={`${t.x}_${t.y}_${t.z}_${i}`}
                xlinkHref={`https://a.tile.openstreetmap.org/${t.z}/${t.x}/${t.y}.png `}
                x={(t.x + this.tiles.translate[0]) * this.tiles.scale}
                y={(t.y + this.tiles.translate[1]) * this.tiles.scale}
                width={this.tiles.scale}
                height={this.tiles.scale}
              />
            ))}
          </g>
        </svg>

        <div class="map__copyright">
          ©&nbsp;
          <a
            href="https://www.openstreetmap.org/copyright"
            target="_blank"
          >OpenStreetMap&nbsp;</a>
          contributors
        </div>
      </div>
    );
  },
};
</script>

<style lang="scss" scoped>
.map {
  position: relative;
  width: 100%;
  height: 100%;
  font-family: Arial, sans, sans-serif;

  &__controls {
    position: absolute;
    left: 16px;
    top: 16px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    height: 56px;
  }
  &__button {
    border: 0;
    padding: 0;
    width: 24px;
    height: 24px;
    line-height: 24px;
    border-radius: 50%;
    font-size: 18px;
    background-color: #ffffff;
    color: #343434;
    box-shadow: 0 1px 4px rgba(0, 0, 0, .4);

    &:hover,
    &:focus {
      background-color: #eeeeee;
    }

    &:disabled {
      background-color: rgba(#eeeeee, .4);
    }
  }
  &__tile {
    pointer-events: none;
  }
  &__copyright {
    position: absolute;
    bottom: 8px;
    right: 8px;
    padding: 2px 4px;
    background-color: rgba(#ffffff, .6);
    font-size: 14px;
  }
}
</style>
