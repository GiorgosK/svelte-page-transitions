<script>
  import Nav from '../components/Nav';
  import PageTransitions from '../components/PageTransitions';
  import { page } from '@sveltejs/kit/assets/runtime/app/stores.js';
</script>

<Nav segment={$page.path}/>

<PageTransitions refresh={$page.path}>
  <slot/>
</PageTransitions>