# Smooth Page transitions in $layout.svelte with SvelteKit (or Sapper)

I have tried many times and I failed to make my page transtitions implemented in a way that we don't have to import the pageTransition component in all routes.

I have finally created a pageTransition component that **can just be dropped in the $layout.svelte** component (_layout.svelte in Sapper) and then be used by all existing or future routes.

I am going to show you here the steps and the thought process for setting up the component and the changes along the way which was the natural steps that I evolved this component and hopefully they can help someone in understanding layout/route based Svelte reactivity better.

Here is the finished demo: <a href="https://amazing-kirch-8cb3f5.netlify.app/" target="_blank">https://amazing-kirch-8cb3f5.netlify.app/</a>

There is also a [github repo](https://github.com/GiorgosK/svelte-page-transitions) that I have committed all different phases if you need to know all the details.  The tutorial is mostly focuses on inexperienced developers. For more experienced developers you can just skip to the last sections or go straight to the repo.

I have chosen SvelteKit for this demonstration just to try it out and mostly because it seems to be the future of Svelte. If you are using Sapper the steps should be almost identical.


## Setup Svelte@next 

```ssh
npm init svelte@next
pnpm install
pnpm run 
```
> NOTE: Feel free to use `npm` where I use `pnpm`. The two have exactly the same syntax.

After that you can browse to `localhost:3000` and be presented with the demo route.

## Setup a 2nd route a Simple Navigation component and a $layout component

Setup minimum routes and components to be able to navigate from one page to another.

```html
<!-- src/components/Nav.svelte -->
<div>
  <a href="/">Home</a>
  <a href="/about">About</a>
</div>
```

```html
<!-- src/routes/about.svelte -->
<main>
  <h1>About Page</h1>
  <p>This is the about page</p>
  <p>More paragraphs of the about page</p>
</main>
```

```html
<!-- src/routes/$layout.svelte -->
<script>
  import Nav from '../components/Nav';
</script>

<Nav/>

<slot/>
```

We should have now the two routes and be able to navigate between Home page and About page.

The beauty of SvelteKit (the magic of snowpack) is that you should be able to see changes in your browser immediately after saving the files.

## Consistent styling of pages by creating global.css

Some stylistic changes are due to make the styles consistent across all pages.

We take all styles from `index.svelte` and add them to a new file `static/global.css` 

```css
/* static/global.css */
:root {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
    Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
}

main {
  text-align: center;
  padding: 1em;
  margin: 0 auto;
}

h1 {
  color: #ff3e00;
  text-transform: uppercase;
  font-size: 4rem;
  font-weight: 100;
  line-height: 1.1;
  margin: 4rem auto;
  max-width: 14rem;
}

p {
  max-width: 14rem;
  margin: 2rem auto;
  line-height: 1.35;
}

@media (min-width: 480px) {
  h1 {
    max-width: none;
  }

  p {
    max-width: none;
  }
}
``` 

Link css file from `app.html` by adding following line before `	%svelte.head%`
```html
<!-- add to src/app.html -->
	<link rel="stylesheet" href="global.css">
```

## Simple PageTransitions component

Here is a simple PageTranstitions svelte component using `svelte/transition` package.

```html
<!-- src/component/PageTransitions.svelte -->
<script>
  import { fly } from 'svelte/transition';
</script>

  <div
    in:fly="{{ y: -50, duration: 250, delay: 300 }}"
    out:fly="{{ y: -50, duration: 250 }}" 
    >
    <slot/>
  </div>
```
It makes the page fly in and out from top of the viewport.  We delay a bit on the in transition to avoid the in and out transitions from working at the same time.  

> Perhaps if anyone know how to add `crossfade` to it please do let me know in the comments.

If you add it as a wrapper on any route it does work as expected and it is the way I have seen all tutorials use a page transition component.  There might also be use cases where we only need it on certain routes and than this would be the way to use it.

```html
<!-- src/routes/about.svelte -->
<script>
	import Counter from '$components/Counter.svelte';
	import PageTransitions from '../components/PageTransitions.svelte';
</script>

<PageTransitions>
  <main>
    <h1>About Page</h1>
    <p>This is the about page</p>
    <p>More paragraphs of the about page</p>
  </main>
</PageTransitions>
```

## More versatile PageTransitions component

For most use cases though I believe setting PageTransitions in the `$layout.svelte` component and than having all routes use it is the way to go as it we set it and forget it.

First lets remove the component from all routes and bring them to their initial state (setup setps a little above).

If we just put the PageTransitions component in `$layout.svelte` component without any modifications though we will not be able to see any transitions.

```html
<!-- src/routes/$layout.svelte -->
<script>
  import Nav from '../components/Nav';
  import PageTransitions from '../components/PageTransitions';
</script>

<Nav/>

<PageTransitions>
  <slot/>
</PageTransitions>
```

> NOTE: I noticed that SvelteKit is not loading all changes automatically to the browser. Perhaps this is something that will be fixed in later versions so lets not focus on it just keep in mind to refresh the browser when you can't see any changes.

The problem is that the component is not reactive as is. We should make it reactive to the change of the route.


### Reactive Nav component
Lets first make the Nav component reactive by underlying the current route link. 

```html
<!-- src/components/Nav.svelte -->
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
```

Add the `segment` variable to `$layout.svelte` to make the Nav reactive.

```html
<!-- src/routes/$layout.svelte -->
<script>
  import Nav from '../components/Nav';
  export let segment;	
</script>

<Nav {segment}/>

<slot/>
```

Now if you change the Nav links the current route link should be underlined.

### Reactive PageTransition component

We now need to make the component reactive by creating a `refresh` prop and using `key` directive which means that when the key changes, svelte removes the component and adds a new one, therefore triggering the transition.

```html
<!-- src/component/PageTransitions.svelte -->
<script>
  import { fly } from 'svelte/transition';
  export let refresh = '';
</script>

{#key refresh}
  <div
    in:fly="{{ y: -50, duration: 250, delay: 300 }}"
    out:fly="{{ y: -50, duration: 250 }}" 
    >
    <slot/>
  </div>
{/key}
```

Also lets pass the `segment` variable to PageTransition component similar to how we did it with the Nav component.

```html
<!-- src/routes/$layout.svelte -->
<script>
  import Nav from '../components/Nav';
  import PageTransitions from '../components/PageTransitions';
  export let segment;	
</script>

<Nav {segment}/>

<PageTransitions refresh={segment}>
  <slot/>
</PageTransitions>
```
You should now have your smooth page transtitions without needing to add the component to every route.

## Improving the page transitions

When the pages are longer in height the transition are not actually that smooth because it might cause the scrollbars to flicker creating an undesired moving effect on the contents of the page.

For understanding the problem lets force each page to be a little bigger in height 

```css
/* add in static/global.css */
main {
  min-height: 600px;
}
```

Depending on your viewport size you might need to adjust the height so that there is no scrollbars on when not transitioning (idle state).

If you refresh your browser and navigate from page to page you should see the flickering effect.

To mitigate this problem lets add the following css to your `global.css`.

```css
/* add in static/global.css */
body {
  overflow-y: scroll;
}
```
And now the scrollbars will be visible at all times but might not always have the scroll handle to drap up/down.  This will prevent them from appearing and disappearing and eliminates the problem.

Another alternative would be to use 

```css
/* add in static/global.css */
html { 
  margin-left: calc(100vw - 100%); } 
}
```

Read more about the flickering scrollbar and the above solutions [on css-tricks](https://css-tricks.com/elegant-fix-jumping-scrollbar-issue/)

## Conclusion

Making a component reactive on the layout/routing level is easy enough as seen above.  This technique can be used with any component we include in `layout` or any component that we want to be reactive when the route changes. 

Please post your thoughts on the comments below.  I would be happy to improve on the above code if something is not considered right or best practice and I do welcome corrections.

If you liked this tutorial or found it useful please share or like or give me a star on [github](https://github.com/GiorgosK/svelte-page-transitions).  Any of the above gives me an incentive to dig deeper and write more useful tutorials.