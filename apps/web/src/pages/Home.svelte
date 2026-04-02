<script>
 import { onMount } from 'svelte';
 import yaml from 'js-yaml';
 import { gte } from '../lib/semver.js';
 import {
     groupBy,
     isEmpty,
     mapValues,
 } from 'lodash-es';

 import {
   EARLIEST_VERSION,
   RELEASES_URL,
   INELIGIBLE_ENDPOINTS_URL,
   PENDING_ENDPOINTS_URL
 } from '../lib/constants.js';

 import {
  releaseJsonExists
 } from '../lib/utils.js';
 import {
   activeFilters,
   activeRelease,
   latestVersion,
   newEndpoints,
   olderNewEndpointsRaw,
   previousVersion,
   releases
 } from '../store';
 import {
   confEndpointsRaw,
   ineligibleEndpoints,
   pendingEndpoints
 } from '../store/conformance.js';
 import Sunburst from '../components/Sunburst/Wrapper.svelte'
 import NewEndpoints from '../components/new-endpoints.svelte';

 let { params, query } = $props();

 let version = $derived(params.version);
 let level = $derived(params.level);
 let category = $derived(params.category);
 let endpoint = $derived(params.endpoint);

 let effectiveVersion = $derived(
   (version === 'latest' || version == null) ? $latestVersion : version
 );

 // Load initial releases data
 $effect(() => {
   if ($releases && isEmpty($releases)) {
     (async () => {
       let releasesFromYaml = await fetch(`${RELEASES_URL}/releases.yaml`)
         .then(res => res.blob())
         .then(blob => blob.text())
         .then(yamlString => yaml.load(yamlString))
         .then(releases => releases.filter(({version}) => gte(version, EARLIEST_VERSION)))

       let releasesData = await releaseJsonExists(releasesFromYaml)
         .then(releases => {
           return mapValues(groupBy(releases, 'version'),
                            ([{version, release_date}]) => ({
                              release: version,
                              release_date: release_date == '' ? new Date() : release_date,
                              spec: '',
                              source: '',
                              endpoints: [],
                              tests: []
           }))
         });
         releases.update(rel => releasesData);
     })();
   }
 });

 // Update active filters when URL params change
 $effect(() => {
     activeFilters.update(af => ({
         ...af,
         version: effectiveVersion,
         level: level || '',
         category: category || '',
         endpoint: endpoint || '',
         conformanceOnly: query["conformance-only"]
                        ? query["conformance-only"].toLowerCase() === "true"
                        : false,
         excludeIneligible: query["exclude-ineligible"]
                        ? query["exclude-ineligible"].toLowerCase() === "true"
                        : false,
         excludePending: query["exclude-pending"]
                        ? query["exclude-pending"].toLowerCase() === "true"
                        : false
     }));
 });

 // Load release-specific endpoint data
 $effect(() => {
     if ($activeRelease && $activeRelease !== 'older' && $activeRelease.endpoints && isEmpty($activeRelease.endpoints)) {
       (async () => {
         let rel = await fetch(`${RELEASES_URL}/${$activeRelease.release}.json`)
             .then(res => res.json());
         releases.update(rels => ({...rels, [$activeRelease.release]: rel}));
       })();
     }
 });

 // Load conformance endpoints
 $effect(() => {
     if ($confEndpointsRaw && isEmpty($confEndpointsRaw)) {
       (async () => {
         const conformanceEndpoints = await fetch(`${RELEASES_URL}/conformance-endpoints.json`)
             .then(res => res.json());
         confEndpointsRaw.set(conformanceEndpoints);
       })();
     }
 });

 // Load ineligible endpoints
 $effect(() => {
     if ($ineligibleEndpoints && isEmpty($ineligibleEndpoints)) {
       (async () => {
         const ineligible = await fetch(INELIGIBLE_ENDPOINTS_URL)
           .then(res => res.text())
           .then(text => yaml.load(text));
         ineligibleEndpoints.set(ineligible);
       })();
     }
 });

 // Load pending endpoints
 $effect(() => {
     if ($pendingEndpoints && isEmpty($pendingEndpoints)) {
       (async () => {
         const pending = await fetch(PENDING_ENDPOINTS_URL)
           .then(res => res.text())
           .then(text => yaml.load(text));
         pendingEndpoints.set(pending);
       })();
     }
 });

 // Load previous version data
 $effect(() => {
     if ($previousVersion !== 'older' && !isEmpty($releases[$previousVersion]) && isEmpty($releases[$previousVersion].endpoints)) {
       (async () => {
         let rel = await fetch(`${RELEASES_URL}/${$previousVersion}.json`)
             .then(res => res.json());
         releases.update(rels => ({...rels, [$previousVersion]: rel}));
       })();
     }
 });

 // Load older new endpoints
 $effect(() => {
     if ($olderNewEndpointsRaw.length === 0) {
       (async () => {
         let older = await fetch(`${RELEASES_URL}/new-endpoints.json`)
             .then(res=>res.json());
         olderNewEndpointsRaw.set(older);
       })();
     }
 });
</script>

<svelte:head>
    <title>APISnoop</title>
</svelte:head>
{#if $activeRelease && $activeRelease.endpoints && $activeRelease.endpoints.length > 0}
    <Sunburst />
    <NewEndpoints />
{:else}
    <em>loading data...</em>
{/if}
