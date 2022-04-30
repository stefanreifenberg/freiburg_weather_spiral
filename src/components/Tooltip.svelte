<script>
    export let x;
    export let y;
    // let's set its placement using [xOffset, yOffset]
    export let placement = [-50, -100];
  
    $: side = placement[1] > -50 ? "bottom" : "top"
    $: arrowTransform = {
      top: "translate(-50%, -100%)",
      bottom: "translate(-50%, 0) rotate(180deg)",
    }[side]
  
  </script>
  
  <div
    class="wrapper"
    style="transform: translate({x}px, {y}px)">
    <div class="tooltip-wrapper"
      style="transform: translate({placement[0]}%, {placement[1]}%)">
      <div class="tooltip">
        <slot />
      </div>
    </div>
    <svg
      class="arrow"
      style="transform: {arrowTransform}">
      <path d="M 0 0 L 6 6 L 12 0" fill="white" stroke="black" />
    </svg>
  </div>
  
  <style>
    .wrapper {
      position: absolute;
      top: 0;
      left: 0;
      z-index: 10;
      /* don't capture mouse events */
      pointer-events: none;
    }
    .tooltip-wrapper {
      position: absolute;
      /* make space for the arrow */
      padding: 5px;
      z-index: 1;
    }
    .arrow {
      position: absolute;
      height: 6px;
      width: 12px;
      /* keep arrow on top, to cover border */
      z-index: 2;
    }
    .tooltip {
      padding: 0.5em;
      background: white;
      min-width: 8em;
      text-align: center;
      font-size: 0.7em;
      border: 1px solid;
      box-shadow: 0 13px 27px -5px rgba(50,50,93,0.25),0 8px 16px -8px rgba(0,0,0,0.3);
    }
  </style>