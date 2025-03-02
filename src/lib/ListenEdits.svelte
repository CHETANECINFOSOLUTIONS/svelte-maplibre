<script lang="ts">
  import { getContext, onMount } from 'svelte';
  import { getMapContext } from './context.svelte';

  const geoman = getContext('geoman');
  const { map } = $derived(getMapContext());
  let { children } = $props();

  // Helper function to safely extract GeoJSON from feature data
  const getGeoJson = (featureData: any) => {
    try {
      return JSON.stringify(featureData.getGeoJson(), null, 2);
    } catch (e) {
      return "Can't retrieve GeoJSON";
    }
  };

  onMount(() => {
    console.log('The geoman', map);
    map.on('gm:create', (e) => {
      console.log('The geoman', e, getGeoJson(e.feature));
      let sourceData = map.getSource('gm_main');
      console.log('The source data', sourceData);
    });
  });
</script>

{@render children?.({})}
