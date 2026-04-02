<script>
 import {
   keyBy
 } from 'lodash-es';
 import {
   breadcrumb,
   mouseOverPath,
   endpoints
 } from '../../store';
 import {
   levelColours,
   categoryColours,
   endpointColour
 } from '../../lib/colours.js';

 let epsByName = $derived(keyBy($endpoints, 'endpoint'));
 let level = $derived($breadcrumb[0]);
 let category = $derived($breadcrumb[1]);
 let endpoint = $derived($breadcrumb[2]);
 let lColour = $derived(levelColours[level] || 'white');
 let cColour = $derived(categoryColours[category] || 'white');
 let eColour = $derived(epsByName[endpoint]
            ? endpointColour(epsByName[endpoint])
            : 'white');
 let eTextColour = $derived(epsByName[endpoint]
                ? epsByName[endpoint]['tested'] ? endpointColour(epsByName[endpoint]) : 'gray'
                : 'white');
</script>

<div id='breadcrumb'>
  {#if $mouseOverPath.length > 0}
    <p>{#if level}<span style='border-color: {lColour}; background-color: {lColour};'>{level}</span>{/if}{#if category}<span style='background-Color: {cColour}; border-color: {cColour};'>{category}</span>{/if}{#if endpoint}<span style='border-color: {eColour}; color: {eTextColour};'> {endpoint}</span>{/if}</p>
  {/if}
</div>

<style>
 div{
   height: 3em;
   grid-column: 1/2;
   margin-bottom: 1em;
 }
 p {
   font-size: 1.3em;
   font-color: aliceblue;
 }
 span {
   margin: 0;
   color: #EEEEEE;
   padding: 0.25em;
   border: 1px solid;
 }
</style>
