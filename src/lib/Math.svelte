<!-- src/lib/Math.svelte (updated) -->
<script>
  import { browser } from '$app/environment';
  import 'katex/dist/katex.min.css';

  let { tex = '', block = false } = $props();

  let container;

  $effect(() => {
    if (!browser || !container || !tex) return;
    import('katex').then(({ default: katex }) => {
      katex.render(tex, container, {
        displayMode: block,
        throwOnError: false
      });
    });
  });
</script>

<span bind:this={container} class:inline={!block} class:block />