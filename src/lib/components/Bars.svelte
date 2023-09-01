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

  const width = 600
  const rowHeight = 50
  const gap = 5

  const color1 = 'pink'
  const color2 = 'lightblue'
  const color3 = '#ff6781'
  const color4 = '#4dc1e7'

  $: height = parsed ? rowHeight * parsed.length + gap * (parsed.length - 1) : 0

  $: max1 = parsed ? Math.max(...parsed.map((values) => parseFloat(values[0]))) : 0
  $: max2 = parsed ? Math.max(...parsed.map((values) => parseFloat(values[1]))) : 0

  const getWidth = (percentage: string | number) => {
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

<div>
  <textarea bind:value={data} />

  {#if parsed}
    <svg
      bind:this={svgElement}
      viewBox="0 0 {width} {height}"
      {width}
      {height}
      xmlns="http://www.w3.org/2000/svg"
    >
      {#each parsed as values, i}
        {@const y = i * (rowHeight + gap)}
        <g transform="translate({getWidth(max1 - parseInt(values[0]))}, 0)">
          <rect x="0" {y} fill={color1} width={getWidth(values[0])} height={rowHeight} />
          <rect
            fill={color2}
            x={getWidth(values[0])}
            {y}
            width={getWidth(values[1])}
            height={rowHeight}
          />
          <rect
            x={getWidth(values[0]) - getWidth(values[2])}
            {y}
            fill={color3}
            width={getWidth(values[2])}
            height={rowHeight}
          />
          <rect
            x={getWidth(values[0])}
            {y}
            fill={color4}
            width={getWidth(values[3])}
            height={rowHeight}
          />
        </g>
      {/each}
    </svg>
  {/if}

  <button on:click={downloadSvg}>Download</button>

  <!-- <pre>{svg}</pre> -->
</div>

<style lang="postcss">
  textarea {
    width: 20rem;
    height: 5rem;
  }
  svg {
    display: block;
    margin: 3rem 0;
  }
  button {
    padding: 1rem 2rem;
    font-size: 2rem;
  }
</style>