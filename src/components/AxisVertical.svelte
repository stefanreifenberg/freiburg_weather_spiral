<script>
    export let scale;
    export let count;
    export let format = d => d;
  
      $: ticks = scale.ticks(count);
  </script>
  
  <line
    y1={scale.range()[0]}
    stroke="black"
  />
  {#each ticks as y}
    <g transform="translate(0, {scale(y)})">
      <text
        x="-10"
        text-anchor="end"
        dominant-baseline="middle">
        {format(y)}
      </text>
  
      <line
        x1="-5"
        stroke="black"
      />
    </g>
  {/each}
  
  <style>
      text {
        font-size: 0.8em;
      }
  </style>