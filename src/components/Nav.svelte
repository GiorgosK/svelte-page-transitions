<script>
	export let segment;
</script>

<style>
  a {
    text-decoration: none;
  }
  .current {
    text-decoration: underline;
  }
</style>
<div>
  <a href="/" class='{segment === undefined ? "current" : ""}'>Home</a>
  <a href="/about" class='{segment === "about" ? "current" : ""}'>About</a>
</div>