<script>
 import EndpointStats from './EndpointStats.svelte';
 import { isEmpty } from 'lodash-es';
 import { currentDepth, breadcrumb, coverageAtDepth } from '../../store';

 let percentage = (sum, total) => ((sum / total) * 100).toFixed(2);
 let total = $derived($coverageAtDepth.totalEndpoints);
 let tested = $derived($coverageAtDepth.testedEndpoints);
 let confTested = $derived($coverageAtDepth.confTestedEndpoints);
 let percentTested = $derived(`${percentage(tested,total)}%`);
 let percentConfTested = $derived(`${percentage(confTested, total)}%`);
 let level = $derived($breadcrumb[0] || '');
 let category = $derived($breadcrumb[1] || '');
 let endpoint = $derived($breadcrumb[2] || '');
</script>

{#if $currentDepth === 'endpoint'}
<EndpointStats />
{:else}
<div id='coverage-stats'>
  <p class='breadcrumb'>{level} {category}</p>
  <h2> Coverage</h2>
  <ul>
    <li><strong>{total}</strong> total endpoints</li>
    <li> <strong>{percentTested}</strong> tested ({tested} endpoints)</li>
    <li><strong>{percentConfTested}</strong> conformance tested ({confTested} endpoints)</li>
  </ul>
</div>
{/if}

<style>
 div {
   grid-column: 2;
   padding-left: 1em;
   padding-right: 1em;
 }

 h2 {
   margin-bottom: 0;
 }

 p {
   margin-top: 0;
   margin-bottom: 0;
   padding: 0;
   font-weight: 200;
   font-size: 1.3em;
   height: 1.5em;
   font-variant-caps: small-caps;
 }


 ul {
   padding-left: 0;
   list-style-type: none;
 }

 strong {
   font-family: monospace;
 }

</style>
