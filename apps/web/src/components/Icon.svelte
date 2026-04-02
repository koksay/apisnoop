<script>
 let { icon, class: className = '' } = $props();
</script>

<svg
  class={className}
  aria-hidden="true"
  xmlns="http://www.w3.org/2000/svg"
  viewBox="0 0 {icon.icon[0]} {icon.icon[1]}"
  width="1em"
  height="1em"
  style="vertical-align: -0.125em;"
>
  <path fill="currentColor" d={icon.icon[4]} />
</svg>
