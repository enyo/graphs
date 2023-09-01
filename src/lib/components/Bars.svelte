<script lang="ts">
  import { parse } from 'csv-parse/browser/esm/sync'
  import { tick } from 'svelte'
  import { download } from './download'

  let data = `40,60,10,5
30,70,4,20
20,80,8,17
70,30,20,4
40,60,4,22
`

  let parsed: string[][] = []

  $: {
    try {
      parsed = data ? parse(data) : []
    } catch (e) {
      parsed = []
      console.error(e)
    }
  }

  let width = 600
  let rowHeight = 50
  let gap = 5

  let color1 = '#ffc0cb'
  let color2 = '#add8e6'
  let color3 = '#ff6781'
  let color4 = '#4dc1e7'

  $: height = parsed ? rowHeight * parsed.length + gap * (parsed.length - 1) : 0

  $: max1 = parsed ? Math.max(...parsed.map((values) => parseFloat(values[0]))) : 0
  $: max2 = parsed ? Math.max(...parsed.map((values) => parseFloat(values[1]))) : 0

  const getWidth = (width: number, percentage: string | number) => {
    const value = typeof percentage === 'number' ? percentage : parseFloat(percentage)
    return (value / (max1 + max2)) * width
  }

  let svgElement: SVGElement

  let svg: string

  const updateSvg = async (element: SVGElement) => {
    await tick()

    svg = element?.outerHTML
  }

  $: parsed && updateSvg(svgElement)

  const downloadSvg = () => {
    const blob = new Blob([svg], { type: 'image/svg+xml' })
    const url = URL.createObjectURL(blob)
    download(url, 'chart.svg')
  }
</script>

Your data:<br />
<textarea bind:value={data} />

{#if parsed}
  <div class="graph">
    <div class="actions">
      <div class="settings">
        <label>
          Color 1: <input type="color" bind:value={color1} />
        </label>
        <label>
          Color 2: <input type="color" bind:value={color2} />
        </label>
        <label>
          Color 3: <input type="color" bind:value={color3} />
        </label>
        <label>
          Color 4: <input type="color" bind:value={color4} />
        </label>
        <label>
          Height: <input type="range" min="10" max="100" bind:value={rowHeight} />
          <input type="number" bind:value={rowHeight} />
        </label>
        <label>
          Width: <input type="range" min="100" max="1000" bind:value={width} />
          <input type="number" bind:value={width} />
        </label>
        <label>
          Gap: <input type="range" min="0" max="30" bind:value={gap} />
          <input type="number" bind:value={gap} />
        </label>
      </div>
      <button on:click={downloadSvg}>Download</button>
    </div>
    <div class="svg">
      <svg
        bind:this={svgElement}
        viewBox="0 0 {width} {height}"
        {width}
        {height}
        xmlns="http://www.w3.org/2000/svg"
      >
        {#each parsed as values, i}
          {@const y = i * (rowHeight + gap)}
          <g transform="translate({getWidth(width, max1 - parseInt(values[0]))}, 0)">
            <rect x="0" {y} fill={color1} width={getWidth(width, values[0])} height={rowHeight} />
            <rect
              fill={color2}
              x={getWidth(width, values[0])}
              {y}
              width={getWidth(width, values[1])}
              height={rowHeight}
            />
            <rect
              x={getWidth(width, values[0]) - getWidth(width, values[2])}
              {y}
              fill={color3}
              width={getWidth(width, values[2])}
              height={rowHeight}
            />
            <rect
              x={getWidth(width, values[0])}
              {y}
              fill={color4}
              width={getWidth(width, values[3])}
              height={rowHeight}
            />
          </g>
        {/each}
      </svg>
    </div>
  </div>
{/if}

<style lang="postcss">
  textarea {
    width: 20rem;
    height: 5rem;
  }
  .svg {
    padding: 1.5rem;
    background: #fafafa;
  }
  svg {
    display: block;
  }
  button {
    padding: 1rem 2rem;
    font-size: 2rem;
  }
  .graph {
    display: flex;
    gap: 2rem;
    flex-wrap: wrap;
    align-items: flex-start;
  }
  .settings {
    margin-bottom: 3rem;
  }
  label {
    display: block;
    margin-top: 0.5rem;
  }
</style>
