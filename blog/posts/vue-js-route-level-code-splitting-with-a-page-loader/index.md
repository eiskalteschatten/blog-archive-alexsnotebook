<figure><img loading="lazy" decoding="async" src="blake-connally-B3l0g6HLxr8-unsplash.jpg" alt=""></figure>

Vue.js combined with Vue Router makes it easy to implement router-level code splitting. For this post, I’m going to assume you know what I am talking about. If you don’t, please see the Vue Router documentation for [Lazy Loading Routers](https://router.vuejs.org/guide/advanced/lazy-loading.html).

When code splitting at the router level, however, it means that the user downloads a new JavaScript file every time the route changes. As the user browses the website, these files will be cached and the browser won’t have to download them again, but the initial load may take a little while before it is finished which is why it is important to give the user feedback to let him or her know that your website is still actually doing something.

That is where page loaders come in. A page loader is nothing more than some sort of indication that a page is loading. They come in many forms from simple text (“Loading…”) to fancy animations. For this example, we are going to use a Google-style loader that will look like this when it is done:

<figure><video height="2236" style="aspect-ratio: 3840 / 2236;" width="3840" controls="" src="https://blog.alexseifert.com/wp-content/uploads/2023/04/vue-page-loader.mov"></video></figure>

Our page loader is an animated bar (in Vue.js’s green!) that runs across the top of the screen. This is a convenient way of displaying a loading status as it is non-blocking and universal.

So how do we do that?

Fortunately, Vue.js makes it easy. I also tried to implement this example using React and React Router, but gave up at after two days of fighting with it. It only took me a couple of hours to implement it in Vue.js and most of it was spent trying to get the bar animation the way I wanted it. The actual logic that shows and hides the page loader was done in only a few minutes.

At this point, if you would just like to get straight into the code without the explanation, see the [GitHub repository](https://github.com/Developers-Notebook/vue-route-code-splitting-page-loader) I created for it.

Code Splitting
--------------

The first thing we need to do is add the code splitting at the route-level by lazy-loading the views. When you create a Vue.js app using their init script, the About page will automatically be lazy-loaded. We will modify this slightly to lazy load the homepage as well using the import() function:

```javascript
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      name: 'home',
      component: () => import('../views/HomeView.vue')
    },
    {
      path: '/about',
      name: 'about',
      // route level code-splitting
      // this generates a separate chunk (About.[hash].js) for this route
      // which is lazy-loaded when the route is visited.
      component: () => import('../views/AboutView.vue')
    }
  ]
})

export default router
```

That is all that needs to be done in order to code-split at the route-level. As you add more routes to your application, you just use the import() function to lazy-load them.

Adding a UI Store
-----------------

Before we get to the page loader component, we need to add the ability to control when the page loader should be shown. To do this, we are going to use Vue.js’s default store, Pinia. In this example, I have created a store called “UI”, but you can put this logic in any existing store you might have depending on what makes sense for your application.

The logic is rudimentary. We are just going to add a basic boolean called “isLoading” to the store:

```javascript
import { ref } from 'vue'
import { defineStore } from 'pinia'

export const useUIStore = defineStore('ui', () => {
  const isLoading = ref(false)

  return {
    isLoading,
  }
})
```

That’s all that needs to be done here.

The Page Loader Component
-------------------------

Next, we need to add the page loader component. This will be a simple component that will contain only the logic needed to shuffle the CSS classes that will control the visibility of the loader:

```javascript
<script setup lang="ts">
import { storeToRefs } from 'pinia'
import { useUIStore } from '@/stores/ui'

const uiStore = useUIStore()
const { isLoading } = storeToRefs(uiStore)
</script>

<template>
  <div
    :class="{
      'page-loader': true,
      loading: isLoading,
      hidden: !isLoading,
    }"
  >
    <div class="bar" />
  </div>
</template>

<style scoped>
.page-loader {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  z-index: 10000000;
  pointer-events: none;
  opacity: 0;
  transition: width 1350ms ease-in-out, opacity 350ms linear, left 50ms ease-in-out;
}

.bar {
  background-color: hsla(160, 100%, 37%, 1);
  height: 5px;
  width: 100%;
}

.hidden {
  opacity: 0;
}

.loading {
  opacity: 1;
  animation: loading 2000ms ease-in-out;
  animation-iteration-count: infinite;
}

@keyframes loading {
  0% {
    width: 0;
    left: 0;
  }
  50% {
    width: 100%;
    left: 0;
  }
  100% {
    width: 100%;
    left: 100%;
  }
}
</style>

```

The most complicated part of this component is the CSS animation, but that is also still fairly basic.

Now that we have the page loader component, we need to add it to our application so that it is globally available. To do this, we can add it to the top of the App.vue file. Here is an abridged version of the file with the relevant parts:

```javascript
<script setup lang="ts">
...
import PageLoader from './components/PageLoader.vue'
...
</script>

<template>
  <PageLoader />
  ...
  <RouterView />
</template>

<style scoped>
...
</style>
```

This is the only file I have shortened since it contains a lot of code unnecessary for this particular example. To view the entire file, take a look at it on [GitHub](https://github.com/Developers-Notebook/vue-route-code-splitting-page-loader/blob/main/src/App.vue).

Gluing It All Together
----------------------

The only piece missing now is the glue that makes it work. Our routes are lazy-loaded, we have prepared our store and created a page loader component. Now, we need to add the logic to control when the page loader component should be shown.

Fortunately, Vue Router makes this very easy for us with so-called [“Navigation Guards”](https://router.vuejs.org/guide/advanced/navigation-guards.html). These are simple functions added to the router object that are called during different stages of a navigation action’s lifecycle.

In this case, we need the “beforeEach” and “afterEach” guards where we will update the “isLoading” variable in the UI store:

```javascript
router.beforeEach(() => {
  const uiStore = useUIStore()
  uiStore.isLoading = true
})

router.afterEach(() => {
  const uiStore = useUIStore()
  uiStore.isLoading = false
})
```

You can place these at the end of the router file that we updated earlier so that the entire file looks something like this:

```javascript
import { createRouter, createWebHistory } from 'vue-router'
import { useUIStore } from '@/stores/ui'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      name: 'home',
      component: () => import('../views/HomeView.vue')
    },
    {
      path: '/about',
      name: 'about',
      // route level code-splitting
      // this generates a separate chunk (About.[hash].js) for this route
      // which is lazy-loaded when the route is visited.
      component: () => import('../views/AboutView.vue')
    }
  ]
})

export default router

router.beforeEach(() => {
  const uiStore = useUIStore()
  uiStore.isLoading = true
})

router.afterEach(() => {
  const uiStore = useUIStore()
  uiStore.isLoading = false
})
```

That’s it! That’s all the needs to be done. You should now have a working, animated page loader that is shown when a route is changed and hidden when the route has finished loading.

Conclusion
----------

Vue.js and Vue Router make it incredibly easy to implement a page loader. There is surprisingly little extra code involved which is due to the fact that Vue Router has Navigation Guards as well as the fact that the view components are not unmounted when lazy loading like they are in React.

You can find the repository with a working example of a page loader here: [https://github.com/Developers-Notebook/vue-route-code-splitting-page-loader](https://github.com/Developers-Notebook/vue-route-code-splitting-page-loader)

*Have you successfully implemented a page loader in Vue.js? How did it vary from my example? Have you implemented one in another library such as React or Angular? How did you solve the problem? Let me know in the comments below!*