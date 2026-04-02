<script>
 import { onMount } from 'svelte';
 import { trimEnd } from 'lodash-es';
 import page from 'page';

 import {
   stableCoverageAtReleaseRaw,
   coverageByReleaseRaw,
 } from '../../store';
 import { RELEASES_URL } from '../../lib/constants.js';

 import Link from '../../components/icons/link-solid.svelte';
 import StableCoverageAtRelease from '../../components/vega-charts/stable-coverage-at-release.svelte';
 import CoverageByRelease from '../../components/vega-charts/coverage-by-release.svelte';

 let { params, query } = $props();

 let stablechart = $derived(query.stablechart || 'number');
 let relchart = $derived(query.relchart || 'number');

 const switchType = ({chart, type}) => {
   let x = window.pageXOffset;
   let y = window.pageYOffset;
   const updatedQuery = {...query, [chart]: type};
   const queryParams = Object.keys(updatedQuery);
   const queryParamsPath = trimEnd(queryParams.map(qp => `${qp}=${updatedQuery[qp]}&`).join(''), '&');
   page(`/conformance-progress?${queryParamsPath}`);
 };

 onMount(async() => {
   let progressData = await fetch(`${RELEASES_URL}/conformance-progress.json`).then(res => res.json());
   let coverageData = await fetch(`${RELEASES_URL}/conformance-coverage-per-release.json`).then(res=>res.json());
   stableCoverageAtReleaseRaw.set(progressData);
   coverageByReleaseRaw.set(coverageData);
 });
</script>

<svelte:head>
    <title>APISnoop | Conformance Progress</title>
</svelte:head>

<h1>Conformance Progress Charts</h1>

<section id="prologue">
  <em>The history of conformance coverage for kubernetes stable endpoints, and where we are at today.</em>
  <p><b>Note:</b> These charts are focused to endpoints that exist today and are eligible for conformance.  Because of this, the numbers will be different from our main page, as that page includes ineligible endpoints.  To see what is excluded, check out <a href="conformance-progress/ineligible-endpoints">our list of ineligible endpoints</a>.</p>
</section>

<StableCoverageAtRelease chartType={stablechart} onChartTypeSwitched={switchType} />
<CoverageByRelease chartType={relchart} onChartTypeSwitched={switchType} />


<style>
 /* h2 {
   margin-top: 2rem;
   font-size: 1.75rem;
 } */

 a:hover {
   background: aliceblue;
 }

</style>
