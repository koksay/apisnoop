<script>
 import { activeFilters, newEndpoints } from '../store';
 import { faCheckCircle } from '@fortawesome/free-solid-svg-icons/faCheckCircle';
 import Icon from './Icon.svelte';
 import { orderBy} from 'lodash-es';
 const checkmark = faCheckCircle;

 let sortedEndpoints = $state([]);
 let sorting = $state('asc');
 let f = $derived($activeFilters);

 $effect(() => {
   sortedEndpoints = $newEndpoints;
 });

 const updateSorting = (key) => {
   sorting = sorting === 'asc'
           ? 'desc'
           : 'asc';
   sortedEndpoints = orderBy(sortedEndpoints, key, sorting);
 }


</script>
<section id="new-endpoints">
<h2>New Endpoints</h2>
<table>
  <thead>
    <tr>
      <th onclick={() => updateSorting('endpoint')}>Endpoint</th>
      <th onclick={() => updateSorting('level')}>Level</th>
      <th onclick={() => updateSorting('category')}>Category</th>
      <th onclick={() => updateSorting('tested')}>Tested</th>
      <th onclick={() => updateSorting('conf_tested')}>Conformance Tested</th>
    </tr>
  </thead>
  <tbody>
    {#each sortedEndpoints as {endpoint, level, category, tested, conf_tested}}
      <tr>
        <td>
          <a href={`/${f.version}/${level}/${category}/${endpoint}`}>{endpoint}</a>
        </td>
        <td>
          <a href={`/${f.version}/${level}`}>
            {level}
          </a>
        </td>
        <td>
          <a href={`/${f.version}/${level}/${category}`}>
            {category}
          </a>
        </td>
        <td>
          {#if tested}
            <Icon class='success check' icon={checkmark} />
          {:else}
            <Icon class='check fail' icon={checkmark} />
          {/if}
        </td>
          <td>
            {#if conf_tested}
              <Icon class='success check' icon={checkmark} />
            {:else}
              <Icon class='check fail' icon={checkmark} />
            {/if}
          </td>
      </tr>
    {/each}
  </tbody>
</table>
</section>
<style>
 section {
   margin-top: 2rem;
 }
 th {
   cursor: pointer;
   padding-right: 1rem;
 }
 th:hover {
   background: aliceblue;
 }
 td {
   padding-right: 1.25rem;
 }
 td :global(.check) {
   font-size: 1.3em;
   padding-right: 0;
   margin-top: 0.1em;
   display: block;
   margin-left: auto;
   margin-right: auto;
 }
 td :global(.success) {
   color: rgba(60, 180, 75, 1);
 }

 td :global(.fail) {
   color: rgba(233, 203, 233, 1);
 }
</style>
