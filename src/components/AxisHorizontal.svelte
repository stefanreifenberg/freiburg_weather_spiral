<script>
    export let scale;
    export let count;
    export let format = d => d;
  
      $: ticks = scale.ticks(count);
  </script>
  
  <line
    x1={scale.range()[1]}
    stroke="black"
  />
  {#each ticks as x}
    <g transform="translate({scale(x)}, 0)">
      <text
        y="10"
        text-anchor="middle"
        dominant-baseline="hanging">
        {format(x)}
      </text>
      <line
        y1="5"
        stroke="black"
      />
    </g>
  {/each}
  
  <style>
      text {
        font-size: 0.8em;
      }
  </style>