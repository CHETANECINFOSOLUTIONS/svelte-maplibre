<script>
  import { getContext, onMount } from 'svelte';

  const geoman = getContext('geoman');
  let { data, children } = $props();

  onMount(() => {
    console.log('The geoman', data);
    geoman.features.importGeoJson(data);
  });
</script>

{@render children?.({ data })}
