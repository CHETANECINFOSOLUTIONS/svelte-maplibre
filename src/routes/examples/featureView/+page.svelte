<script lang="ts">
  import MapLibre from '$lib/MapLibre.svelte';
  import GeoJSON from '$lib/GeoJSON.svelte';
  import FillLayer from '$lib/FillLayer.svelte';
  import Control from '$lib/Control.svelte';
  import { bbox, getType } from '@turf/turf';
  import GeomanGeoJson from '$lib/GeomanGeoJSON.svelte';
  import Popup from '$lib/Popup.svelte';
  import ListenEdits from '$lib/ListenEdits.svelte';
  import DefaultMarker from '$lib/DefaultMarker.svelte';
  import MarkerLayer from '$lib/MarkerLayer.svelte';
  import flush from 'just-flush';

  let geoman = $state();
  let map: maplibregl.Map = $state();
  let addGeoMan = $state(true);

  function getAndClearSourceLayers(sourceId: string) {
    if (!map.getSource(sourceId)) return null; // Check if source exists

    // Step 1: Get the current GeoJSON data from the source
    let sourceData = map.getSource(sourceId)._data;

    console.log('The source data', sourceData);

    // Step 2: Filter features that are not null and have "gm_main" in their id
    sourceData.features = sourceData.features.filter((feature) => feature !== null);

    // Step 4: Clear the source data by setting an empty FeatureCollection
    map.getSource(sourceId).setData({
      type: 'FeatureCollection',
      features: [],
    });

    geoman.features.forEach((f: any) => {
      geoman.features.delete(f);
      console.log('is available', geoman.features.has(f));
    });
    // console.log('the layers ', geoman.features);

    return sourceData; // Return cleaned GeoJSON
  }

  let FC = $state({
    type: 'FeatureCollection',
    features: [
      {
        type: 'Feature',
        properties: {
          name: 'Polygon 1',
          paint: {
            'fill-color': '#ff0000',
            'fill-opacity': 0.5,
          },
        },
        geometry: {
          type: 'Polygon',
          coordinates: [
            [
              [77.0, 28.0],
              [77.01, 28.0],
              [77.01, 28.01],
              [77.0, 28.01],
              [77.0, 28.0],
            ],
          ],
        },
      },
      {
        type: 'Feature',
        properties: {
          name: 'Polygon 2',
          paint: {
            'fill-color': '#0000ff',
            'fill-opacity': 0.5,
          },
        },
        geometry: {
          type: 'Polygon',
          coordinates: [
            [
              [77.012, 28.0],
              [77.022, 28.0],
              [77.022, 28.01],
              [77.012, 28.01],
              [77.012, 28.0],
            ],
          ],
        },
      },
    ],
  });

  let bounds = $derived.by(() => {
    return bbox(FC);
  });

  $inspect(console.log('The FC', FC.features));

  let editFeatures = $state(false);
</script>

<MapLibre
  style="https://basemaps.cartocdn.com/gl/positron-gl-style/style.json"
  bind:map
  bind:geoman
  {addGeoMan}
  class="relative aspect-[9/16] max-h-[70vh] w-full sm:aspect-video sm:max-h-full"
  standardControls
  {bounds}
>
  <ListenEdits />
  {#if !editFeatures}
    {#each FC.features as ft, index}
      {@const { name, paint } = ft.properties}
      {@const geomType = getType(ft)}
      {#if geomType !== 'Point'}
        <GeoJSON data={ft} id={`${index}`} promoteId="name">
          <FillLayer
            paint={{
              'fill-color': '#0000ff',
              'fill-opacity': 0.5,
            }}
          >
            <Popup>
              <button>Edit</button>
            </Popup>
          </FillLayer>
        </GeoJSON>
      {:else if geomType === 'Point'}
        <DefaultMarker lngLat={ft.geometry.coordinates} draggable>
          <Popup>
            <button>Edit</button>
          </Popup>
        </DefaultMarker>
      {/if}
    {/each}
  {:else if editFeatures}
    {#each FC.features as ft}
      <GeomanGeoJson data={ft}></GeomanGeoJson>
    {/each}
  {/if}
  <Control>
    <button
      class=" right-0 top-0 m-1 rounded-lg bg-white p-2 text-black shadow-md"
      onclick={() => {
        editFeatures = !editFeatures;
      }}
    >
      Edit Features
    </button>
  </Control>
  {#if editFeatures}
    <Control>
      <button
        class="right-0 top-0 m-1 rounded-lg bg-white p-2 text-black shadow-md"
        onclick={() => {
          editFeatures = false;
          FC = getAndClearSourceLayers('gm_main', map);
        }}
      >
        Save
      </button>
    </Control>
  {/if}
</MapLibre>
