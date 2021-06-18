<script context="module">
	export const load = async ({ page }) => ({
		props: {
			key: page.path
		}
	});
</script>

<script>
	import Nav from '../components/Nav.svelte';
	import PageTransitions from '../components/PageTransitions.svelte';
	import { page } from '$app/stores';
	export let key;
</script>

<Nav segment={key} />

<PageTransitions refresh={key}>
  <slot />
</PageTransitions>
