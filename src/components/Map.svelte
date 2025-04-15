<script>
    // Map.svelte
    import { onMount } from 'svelte';
    import 'ol/ol.css';
    
    /** @type {import('$lib/types').GeoLocation|null} */
    export let location = null;
    
    /** @type {import('ol/Map').default|null} */
    let map = null;
    
    onMount(() => {
      import('ol/Map').then(({ default: Map }) => {
        import('ol/View').then(({ default: View }) => {
          import('ol/layer/Tile').then(({ default: TileLayer }) => {
            import('ol/source/OSM').then(({ default: OSM }) => {
              map = new Map({
                target: 'map',
                layers: [
                  new TileLayer({
                    source: new OSM(),
                    opacity: 1,
                    visible: true
                  })
                ],
                view: new View({
                  center: [0, 0],
                  zoom: 2
                })
              });
              
              if (location) {
                updateMapLocation(location);
              }
            });
          });
        });
      });
      
      return () => {
        if (map) {
          map.setTarget(undefined);
        }
      };
    });
    
    /**
     * @param {import('$lib/types').GeoLocation} loc
     */
    function updateMapLocation(loc) {
      if (!map) return;
      
      import('ol/proj').then(({ fromLonLat }) => {
        import('ol/Feature').then(({ default: Feature }) => {
          import('ol/geom/Point').then(({ default: Point }) => {
            import('ol/layer/Vector').then(({ default: VectorLayer }) => {
              import('ol/source/Vector').then(({ default: VectorSource }) => {
                import('ol/style/Style').then(({ default: Style }) => {
                  import('ol/style/Icon').then(({ default: Icon }) => {
                    // Create a marker for the location
                    const marker = new Feature({
                      geometry: new Point(fromLonLat([loc.longitude, loc.latitude]))
                    });
                    
                    marker.setStyle(new Style({
                      image: new Icon({
                        src: '/marker.png',
                        scale: 0.8
                      })
                    }));
                    
                    const vectorSource = new VectorSource({
                      features: [marker]
                    });
                    
                    const vectorLayer = new VectorLayer({
                      source: vectorSource
                    });
                    
                    map.addLayer(vectorLayer);
                    map.getView().setCenter(fromLonLat([loc.longitude, loc.latitude]));
                    map.getView().setZoom(13);
                  });
                });
              });
            });
          });
        });
      });
    }
    
    $: if (location && map) {
      updateMapLocation(location);
    }
    </script>
    
    <div id="map" style="width: 100%; height: 400px;"></div>
    