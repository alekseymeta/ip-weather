<template>
  <div id="map" class="map__wrap shadow"></div>
</template>

<script>
import mapboxgl from 'mapbox-gl';

export default {
  name: "MapBox",
  props: {
    zoom: {
      type: String,
      default: 12.5
    }
  },
  mounted() {

    mapboxgl.accessToken = process.env.VUE_APP_MAPBOX_TOKEN;

    fetch('https://ipinfo.io?token=' + process.env.VUE_APP_IP_TOKEN)
        .then(resp => resp.json())
        .then(data => {

          const feature = (data.loc).split(',', 3).reverse()

          console.log(feature)


          const map = new mapboxgl.Map({
            container: 'map',
            style: 'mapbox://styles/mapbox/dark-v10',
            center: feature,
            zoom: this.zoom,
            pitch: 60
          });


          function rotateCamera(timestamp) {
            map.rotateTo((timestamp / 100) % 360, {duration: 0});
            requestAnimationFrame(rotateCamera);
          }


          map.on('load', () => {
            rotateCamera(0);
            const layers = map.getStyle().layers;
            const labelLayerId = layers.find(
                (layer) => layer.type === 'symbol' && layer.layout['text-field']
            ).id;
          });

          // new mapboxgl.Marker().setLngLat(feature).addTo(map);

          const size = 100;

          const pulsingDot = {
            width: size,
            height: size,
            data: new Uint8Array(size * size * 4),

            onAdd: function () {
              const canvas = document.createElement('canvas');
              canvas.width = this.width;
              canvas.height = this.height;
              this.context = canvas.getContext('2d');
            },

            render: function () {
              const duration = 1000;
              const t = (performance.now() % duration) / duration;

              const radius = (size / 2) * 0.3;
              const outerRadius = (size / 2) * 0.7 * t + radius;
              const context = this.context;

              context.clearRect(0, 0, this.width, this.height);
              context.beginPath();
              context.arc(
                  this.width / 2,
                  this.height / 2,
                  outerRadius,
                  0,
                  Math.PI * 2
              );
              context.fillStyle = `rgba(255, 200, 200, ${1 - t})`;
              context.fill();

              context.beginPath();
              context.arc(
                  this.width / 2,
                  this.height / 2,
                  radius,
                  0,
                  Math.PI * 2
              );
              context.fillStyle = 'rgb(140,78,246)';
              context.strokeStyle = 'white';
              context.lineWidth = 2 + 4 * (1 - t);
              context.fill();
              context.stroke();

              this.data = context.getImageData(
                  0,
                  0,
                  this.width,
                  this.height
              ).data;
              map.triggerRepaint();
              return true;
            }
          };

          map.on('load', () => {
            map.addImage('pulsing-dot', pulsingDot, {pixelRatio: 2});

            map.addSource('dot-point', {
              'type': 'geojson',
              'data': {
                'type': 'FeatureCollection',
                'features': [
                  {
                    'type': 'Feature',
                    'geometry': {
                      'type': 'Point',
                      'coordinates': feature // icon position [lng, lat]
                    }
                  }
                ]
              }
            });
            map.addLayer({
              'id': 'add-3d-buildings',
              'source': 'composite',
              'source-layer': 'building',
              'filter': ['==', 'extrude', 'true'],
              'type': 'fill-extrusion',
              'minzoom': 12,
              'paint': {
                'fill-extrusion-color': '#aaa',
                'fill-extrusion-height': [
                  'interpolate',
                  ['linear'],
                  ['zoom'],
                  12,
                  0,
                  15.05,
                  ['get', 'height']
                ],
                'fill-extrusion-base': [
                  'interpolate',
                  ['linear'],
                  ['zoom'],
                  12,
                  0,
                  15.05,
                  ['get', 'min_height']
                ],
                'fill-extrusion-opacity': 0.6
              }
            });
            map.addLayer({
              'id': 'layer-with-pulsing-dot',
              'type': 'symbol',
              'source': 'dot-point',
              'layout': {
                'icon-image': 'pulsing-dot'
              }
            });


            // map.scrollZoom.disable();
            map.setLayoutProperty('country-label', 'city-label', 'text-field', [
              'get',
              `name_Russian`
            ]);

          });
        })

    }

}
</script>

<style scoped>
.map__wrap {
  height: 100%;
  filter: brightness(0.4);
}

</style>