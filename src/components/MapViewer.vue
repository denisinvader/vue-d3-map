<script>
const d3 = {
  ...require('d3-geo'),
  ...require('d3-tile'),
};

export default {
  props: {
    center: {
      type: Array,
      default: () => [-7.584838, 33.561041],
    },
    scale: {
      type: [Number, String],
      default: 1 << 20,
    },
  },
  data () {
    return {
      width: 0,
      height: 0,
    };
  },
  computed: {
    projection () {
      return d3.geoMercator()
        .scale(+this.scale / (2 * Math.PI))
        .translate([this.width / 2, this.height / 2])
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
  mounted () {
    const rect = this.$el.getBoundingClientRect();

    this.width = rect.width;
    this.height = rect.height;
  },
  render () {
    if (this.width <= 0 || this.height <= 0) {
      return <div class="map" />;
    }

    return (
      <div class="map">
        <svg viewBox={`0 0 ${this.width} ${this.height}`}>
          <g>
            {this.tiles.map(t => (
              <image
                key={`${t.x}_${t.y}_${t.z}`}
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
