<script>
    export let heightScale
    export let colorScale
  
    $: ticks = heightScale.ticks(4)
  </script>
  
  <div class="wrapper">
    <div class="ticks">
      {#each ticks as value, i}
        <div class="tick" style="
          height: {heightScale.range()[1] + 25}px;
        ">
          <div
            class="tick-line"
            style="
              height: {heightScale(value)}px;
              background: {colorScale(value)};
            "
          />
          {#if !(i % 2)}
            <div class="tick-value">
              {value}°C
            </div>
          {/if}
        </div>
      {/each}
    </div>
    <div class="label">
      max temperature
    </div>
  </div>
  
  <style>
    .wrapper {
      position: absolute;
      /* centered in the viz */
      /* https://wattenberger.com/blog/css-percents */
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      font-size: 1em;
      font-family: monospace;
      display: flex;
      flex-direction: column;
      align-items: center;
    }
    .ticks {
      display: flex;
      align-items: center;
    }
    .tick {
      position: relative;
      padding: 10px;
    }
    .tick-line {
      position: absolute;
      transform: translate(-50%, -50%);
      top: 50%;
      width: 6px;
      border-radius: 6px;
    }
    .tick-value {
      position: absolute;
      bottom: 0;
      left: 50%;
      transform: translate(-50%, -50%);
    }
  </style>