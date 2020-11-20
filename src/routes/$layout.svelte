<script>
  import Nav from '../components/Nav';
  import PageTransitions from '../components/PageTransitions';
  export let segment;	
</script>

<Nav {segment}/>

<PageTransitions refresh={segment}>
  <slot/>
</PageTransitions>