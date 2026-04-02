<script>
 import * as d3 from 'd3';
 import page from "page";
 import {
   compact,
   join
 } from 'lodash-es';
 import {
   activeRelease,
   activeFilters,
   mouseOverPath,
   zoomedSunburst
 } from '../../store';

 const format = d3.format(",d")
 const width = 932
 const radius = width / 8
 const arc = d3.arc()
               .startAngle(d => d.x0)
               .endAngle(d => d.x1)
               .padAngle(d => Math.min((d.x1 - d.x0) / 2, 0.005))
               .padRadius(radius * 1.5)
               .innerRadius(d => d.y0 * radius)
               .outerRadius(d => Math.max(d.y0 * radius, d.y1 * radius - 1))

 let activeDepth = $derived(determineDepth($activeFilters));

 function determineDepth (filters) {
   let { level, category, endpoint } = filters;
   let setFilters = compact([level, category, endpoint])
   if (setFilters.length === 3) {
     return 'endpoint'
   } else if (setFilters.length === 2) {
     return 'category';
   } else if (setFilters.length === 1) {
     return 'level';
   } else {
     return 'root'
   }
 };

 function depthUp () {
   let {
     release: version
   } = $activeRelease;
   $mouseOverPath = [];
   if (activeDepth === 'root') {
     return null
   } else if (activeDepth === 'endpoint') {
     activeFilters.update(af => ({...af, endpoint: '', category: ''}));
   } else {
     activeFilters.update(af => ({...af, [activeDepth]: ''}));
   }
   let {level, category, endpoint, conformanceOnly} = $activeFilters;
   let newPath = "/" + compact([version,level,category,endpoint]).join('/');
   if (conformanceOnly) {
     newPath = newPath + "?conformance-only=true"
   }
   page.redirect(newPath);
 };

 function labelVisible(d) {
   return d.y1 <= 3 && d.y0 >= 1 && (d.y1 - d.y0) * (d.x1 - d.x0) > 0.03;
 }

 function labelTransform(d) {
   const x = (d.x0 + d.x1) / 2 * 180 / Math.PI;
   const y = (d.y0 + d.y1) / 2 * radius;
   return `rotate(${x - 90}) translate(${y},0) rotate(${x < 180 ? 0 : 180})`;
 }

 function mouseOver(d) {
   $mouseOverPath = d.ancestors().reverse().slice(1);
 }

 function mouseLeave () {
   $mouseOverPath = [];
 }

 function clicked (node, e) {
   let {
     release: version
   } = $activeRelease;
   let {
     level,
     category,
     endpoint
   } = node.data;
   let { conformanceOnly } = $activeFilters;
   activeFilters.update(af=> ({...af, version, level, category, endpoint}));
   let newPath = "/" + compact([version,level,category,endpoint]).join('/');
   if (conformanceOnly) {
     newPath = newPath + "?conformance-only=true";
   }
   page.redirect(newPath);
 };

 function partitionData(data) {
   const root = d3.hierarchy(data)
                  .sum(d => d.value)
                  .sort((a, b) => (b.data.tested - a.data.tested))
                  .sort((a, b) => (b.data.conf_tested - a.data.tested));
   return d3.partition()
            .size([2 * Math.PI, root.height + 1])
   (root);
 }

 let root = $derived.by(() => partitionData($zoomedSunburst).each(d=> d.current = d));
 let nodes = $derived.by(() => root
   .descendants()
   .slice(1)
   .map((node) => {
     let currentOpacity = 1;
     if ($activeFilters.endpoint !== '' && node.data.endpoint !== '') {
       currentOpacity = ($activeFilters.endpoint === node.data.name)
                      ? 1
                      : 0.3
     }
     if ($mouseOverPath.length > 0) {
       currentOpacity = ($mouseOverPath.indexOf(node) >= 0 || $activeFilters.endpoint === node.data.name)
                      ? 1
                      : 0.3
     }
     return {...node, currentOpacity};
   }));
</script>
<div class="chart">
  <!-- svelte-ignore a11y_no_static_element_interactions -->
  <svg viewBox="0,0,932,932" style="font: 12px sans-serif;" onmouseleave={mouseLeave} id='sunburst'>
    <g transform="translate({width/2},{width/2})" id='big-g' style="cursor:pointer;">
      <g>
        {#each nodes as node}
        <!-- svelte-ignore a11y_no_static_element_interactions -->
        <path
          fill={node.data.color}
          fill-opacity={node.currentOpacity}
          d={arc(node.current)}
          onmouseover={() => mouseOver(node.current)}
          onfocus={() => mouseOver(node.current)}
          style="cursor: pointer;"
          onmousedown={(e)=> clicked(node, e)} />
        {/each}
      </g>
      <g pointer-events='none' text-anchor='middle' style='user-select: none;'>
        {#each nodes as node}
        <text
          dy='0.35em'
          fill-opacity = {+labelVisible(node.current)}
          transform = {labelTransform(node.current)}
        >
          {node.children ? node.data.name : ''}
        </text>
        {/each}
      </g>
      <!-- svelte-ignore a11y_no_static_element_interactions -->
      <circle
        r={radius}
        fill={root.data.color}
        pointer-events="all"
        onclick={depthUp}
        onkeydown={depthUp}
      />

      <text
        text-anchor='middle'
        font-size='2em'
        fill='white'
        transform={$activeFilters.category.length > 0 ? "translate(0, -15)" : ""} >
        {$activeFilters.level}
      </text>
      <text
        text-anchor='middle'
        font-size='2em'
        fill='white'
        transform="translate(0,15)">
        {$activeFilters.category}
      </text>
    </g>
  </svg>
</div>

<style>
 .chart {
   position: relative;
   grid-column: 1;
 }
</style>
