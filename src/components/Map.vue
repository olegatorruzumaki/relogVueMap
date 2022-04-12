<template>
  <div>
    <div id="map"></div>
  </div>
</template>

<script>
import appsJson from "../assets/json/apps.json"
import clientsJson from "../assets/json/clients.json"
import mapboxgl from "mapbox-gl";
import {eventBus} from '@/main'

export default {
  name: 'Main',
  components: {},
  props: {},
  data() {
    return {
      appGeoJson: {"type": "FeatureCollection", "features": []},
      map: null,
    }
  },
  methods: {
    flyTo(long, lat) {
      this.map.flyTo({
        center: [
          long,
          lat
        ],
        zoom: 18,
        essential: true
      })
    },
    getMap() {
      for (let point of appsJson) {
        let coordinate = [parseFloat(point.coords.long), parseFloat(point.coords.lat)];
        let properties = point;
        delete properties.longitude;
        delete properties.latitude;
        let feature = {
          "type": "Feature",
          "geometry": {"type": "Point", "coordinates": coordinate},
          "properties": properties
        }
        this.appGeoJson.features.push(feature);
      }

      this.map.on('load', () => {
        this.map.addSource('apps', {
          type: 'geojson',
          data: this.appGeoJson,
          cluster: true,
          clusterMaxZoom: 14,
          clusterRadius: 50
        });

        this.map.addLayer({
          id: 'clusters',
          type: 'circle',
          source: 'apps',
          filter: ['has', 'point_count'],
          paint: {
            'circle-color': [
              'step',
              ['get', 'point_count'],
              '#51bbd6',
              100,
              '#f1f075',
              750,
              '#f28cb1'
            ],
            'circle-radius': [
              'step',
              ['get', 'point_count'],
              20,
              100,
              30,
              750,
              40
            ]
          }
        });

        this.map.addLayer({
          id: 'cluster-count',
          type: 'symbol',
          source: 'apps',
          filter: ['has', 'point_count'],
          layout: {
            'text-field': '{point_count_abbreviated}',
            'text-font': ['DIN Offc Pro Medium', 'Arial Unicode MS Bold'],
            'text-size': 12
          }
        });

        this.map.addLayer({
          id: 'unclustered-point',
          type: 'circle',
          source: 'apps',
          filter: ['!', ['has', 'point_count']],
          paint: {
            'circle-color': 'red',
            'circle-radius': 8,
            'circle-stroke-width': 1,
            'circle-stroke-color': '#fff'
          }
        });

        this.map.on('click', 'clusters', (e) => {
          const features = this.map.queryRenderedFeatures(e.point, {
            layers: ['clusters']
          });
          this.map.getSource('apps').getClusterExpansionZoom(
              (err, zoom) => {
                if (err) return;

                this.map.easeTo({
                  center: features[0].geometry.coordinates,
                  zoom: zoom
                });
              }
          );
        });

        this.map.on('mouseenter', 'clusters', () => {
          this.map.getCanvas().style.cursor = 'pointer';
        });
        this.map.on('mouseleave', 'clusters', () => {
          this.map.getCanvas().style.cursor = '';
        });

        const popup = new mapboxgl.Popup({
          closeButton: false,
          closeOnClick: false
        });

        this.map.on('mouseenter', 'unclustered-point', (e) => {
          this.map.getCanvas().style.cursor = 'pointer';

          const coordinates = e.features[0].geometry.coordinates.slice();
          while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
            coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
          }

          popup
              .setLngLat(coordinates)
              .setHTML(
                  `ID заказа: ${e.features[0].properties.id}
                        <br>ID клиента: ${clientsJson.find(client => client.id === e.features[0].properties.client_id).name}
                        <br>Цена заказа: ${e.features[0].properties.price}`
              )
              .addTo(this.map);
        });
        this.map.on('mouseleave', 'unclustered-point', () => {
          this.map.getCanvas().style.cursor = '';
          popup.remove();
        });
      });
    }
  },
  mounted() {
    mapboxgl.accessToken = 'pk.eyJ1Ijoib2xlZ2t1cmJhdG92IiwiYSI6ImNsMHMwcGs2dDAyYXUza281ZWZzOTMweTEifQ.qQmKfhReh4wtcW38MCh_qw';
    this.map = new mapboxgl.Map({
      container: 'map',
      style: 'mapbox://styles/mapbox/streets-v11',
      center: [76.889709, 43.238949],
      zoom: 11
    });

    eventBus.$on('flyTo', data => {
      this.flyTo(data.long, data.lat)
    })
    this.getMap();
  },
}

</script>

<style lang="scss" scoped>
</style>
