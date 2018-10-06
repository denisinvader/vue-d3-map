<script>
import MapViewer from './components/MapViewer.vue';

export default {
  name: 'app',
  render () {
    return (
      <MapViewer />
    );
  },
}
</script>

<style>
body {
  margin: 0;
  padding: 0;
  cursor: default;
  width: 100vw;
  height: 100vh;
}
</style>
