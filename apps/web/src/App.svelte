<script>
 import { gte,lte,lt, gt } from './lib/semver.js';
 import router from "page";
 import Home from './pages/Home.svelte';
 import About from './pages/About.svelte';
 import ConformanceProgress from './pages/conformance-progress/Index.svelte';
 import ConformanceEndpoints from './pages/conformance-progress/Endpoints.svelte';
 import IneligibleEndpoints from './pages/conformance-progress/IneligibleEndpoints.svelte';
 import PendingEndpoints from './pages/conformance-progress/PendingEndpoints.svelte';
 import Nav from './components/Nav.svelte';
 import { flatten } from 'lodash-es';

 let Page = $state(null);
 let params = $state({});
 let query = $state({});
 let segment = $state('');

 $effect(() => {
   Page;
   segment = window.location.pathname.split('/')[1];
 });

 function queryObj (qs) {
   // convert query string to object.
   // does not account for arrays at the moment.
   if (qs === "") {
     return { };
   } else {
     const parts = qs.split('&');
     return parts.reduce((acc, cur) => {
       const [k, v] = cur.split("=");
       if (acc[k] != null) {
         acc[k] = flatten([acc[k], v]);
       } else {
         acc[k] = v;
       }
       return acc;
     }, { });
   }
 }

 router("/about", (ctx, next)=> {
   params = ctx.params;
   query = {...queryObj(ctx.querystring)};
   next()}, () => Page = About);

 router("/conformance-progress", (ctx, next) => {
   params = ctx.params;
   query = {...queryObj(ctx.querystring)};
   next()}, () => Page = ConformanceProgress);
 router("/conformance-progress/endpoints/:release?/:filter?", (ctx, next) => {
   params = ctx.params;
   query = {...queryObj(ctx.querystring)};
   next()}, () => Page = ConformanceEndpoints);
 router("/conformance-progress/ineligible-endpoints", () => Page = IneligibleEndpoints);
 router("/conformance-progress/pending-endpoints", () => Page = PendingEndpoints);

 router('/:version?/:level?/:category?/:endpoint?', (ctx, next) => {
   params = ctx.params;
   query = {...queryObj(ctx.querystring)};
   next()},  () => Page = Home);

 router.start();
</script>

<svelte:head>
	<title>APISnoop</title>
</svelte:head>
<Nav {segment}/>
<main>
    {#if Page}
      <Page {params} {query}/>
    {/if}
</main>

<style>

 main {
   position: relative;
   font-size: 16px;
   background-color: white;
   padding: 2em;
   box-sizing: border-box;
   max-width: 1100px;
 }

</style>
