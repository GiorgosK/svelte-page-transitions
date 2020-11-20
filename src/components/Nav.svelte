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
  .float-right {
    float: right;
    padding: 10px;
  }
</style>
<div>
  <a href="/" class='{segment === undefined ? "current" : ""}'>Home</a>
  <a href="/about" class='{segment === "about" ? "current" : ""}'>About</a>
  <a href="https://github.com/GiorgosK/svelte-page-transitions" class="float-right">GitHub Repo</a>
</div>