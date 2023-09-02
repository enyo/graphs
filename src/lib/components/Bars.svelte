<script lang="ts">
  import { parse } from 'csv-parse/browser/esm/sync'
  import { tick } from 'svelte'
  import { download } from './download'
  import { text } from '@sveltejs/kit'

  let data = `40%	60%	10%	5%
30,4%	69,6%	4,0%	20,0%
20,9%	79,1%	8,0%	17,0%
70,0%	30,0%	20,0%	4,0%
40,0%	60,0%	4,0%	22,0%
`

  let parsed: number[][] = []

  $: {
    try {
      parsed = data
        ? parse(data, { delimiter: data.match(/\t/) ? '\t' : ',' }).map((values: string[]) =>
            values.map((value) => {
              if (value.match(/,/)) {
                value = value.replaceAll(/\./g, '').replaceAll(/,/g, '.')
              }

              return parseFloat(value)
            }),
          )
        : []
    } catch (e) {
      parsed = []
      console.error(e)
    }
  }

  let width = 600
  const legendWidth = 200
  let rowHeight = 50
  let gap = 5

  let color1 = '#ffc0cb'
  let color2 = '#add8e6'
  let color3 = '#ff6781'
  let color4 = '#4dc1e7'

  $: height = parsed ? rowHeight * parsed.length + gap * (parsed.length - 1) : 0

  $: max1 = parsed ? Math.max(...parsed.map((values) => values[0])) : 0
  $: max2 = parsed ? Math.max(...parsed.map((values) => values[1])) : 0

  const getWidth = (width: number, percentage: number) => {
    return (percentage / (max1 + max2)) * width
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
  const displayNumber = (num: number) => Math.round(num * 10) / 10
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
        viewBox="0 0 {width + legendWidth} {height}"
        width={width + legendWidth}
        {height}
        xmlns="http://www.w3.org/2000/svg"
      >
        <style>
          text {
            font-family: sans-serif;
            font-weight: bold;
            font-size: 0.875rem;
          }
        </style>
        {#each parsed as values, i}
          {@const y = i * (rowHeight + gap)}
          <g transform="translate({getWidth(width, max1 - values[0])}, {y})">
            <rect x="0" y="0" fill={color1} width={getWidth(width, values[0])} height={rowHeight} />
            <rect
              fill={color2}
              x={getWidth(width, values[0])}
              y="0"
              width={getWidth(width, values[1])}
              height={rowHeight}
            />
            <rect
              x={getWidth(width, values[0]) - getWidth(width, values[2])}
              y="0"
              fill={color3}
              width={getWidth(width, values[2])}
              height={rowHeight}
            />
            <rect
              x={getWidth(width, values[0])}
              y="0"
              fill={color4}
              width={getWidth(width, values[3])}
              height={rowHeight}
            />
          </g>
          <g transform="translate({width + 20}, {y + rowHeight / 2})">
            <text x="0" y={0} dominant-baseline="middle" fill={color1}>
              {displayNumber(values[0])}%
            </text>
            <text x={legendWidth / 4} y={0} dominant-baseline="middle" fill={color2}>
              {displayNumber(values[1])}%
            </text>
            <text x={(2 * legendWidth) / 4} y={0} dominant-baseline="middle" fill={color3}>
              {displayNumber(values[2])}%
            </text>
            <text x={(3 * legendWidth) / 4} y={0} dominant-baseline="middle" fill={color4}>
              {displayNumber(values[3])}%
            </text>
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
