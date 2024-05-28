<script>
  import Scrolly from "./Scrolly.svelte";
  import NormalDist from "./NormalDistribution.svelte";
  import NormalDistQ from "./NormalDistributionQuantile.svelte";
  import katexify from "../katexify";
  import { select, selectAll } from "d3-selection";

  // scroll iterator
  let value;

  // Paragraph text for scrolly
  $: steps = [
    `<h1 class='step-title'>Adjust Mean and Standard Deviation</h1>
      <p>
        What happens if we change the average height or the variation in heights? Use our interactive model to adjust the mean (average height) and standard deviation (variability of height) of our dataset. As you modify these parameters, observe how the shape of the normal distribution changes. This hands-on experience helps to understand the impact of these parameters on the population distribution.
      </p>`,
    `<h1 class='step-title'>Where Do You Stand?</h1>
      <p>
        Curious about how a specific height measures up within a population? Enter any height into our model, and see not only where it falls within the distribution, but also the percentile rank of this height in the population. The graph will highlight the area under the curve up to this height, providing a visual and intuitive understanding of percentiles in a normal distribution.
        </p>
  `,
  ];

  const target2event = {
    0: () => {
      // console.log('0' )
    },
    1: () => {
      NormalDist;
    },

    2: () => {
      NormalDistQ;
    },
  };

  $: if (typeof value !== "undefined") target2event[value]();
</script>

<h2 class="body-header">Side Scrolly Example</h2>
<p class="body-text">
  Here's an example of a typical side-scroller. It's responsive, and will fold
  to an overlap scroll if the screen gets small enough:
</p>
<section>
  <!-- scroll container -->
  <div class="section-container">
    <div class="steps-container">
      <Scrolly bind:value>
        {#each steps as text, i}
          <div class="step" class:active={value === i}>
            <div class="step-content">{@html text}</div>
          </div>
        {/each}
        <div class="spacer" />
      </Scrolly>
    </div>
    <div class="charts-container">
      {#if value === 0}
        <NormalDist />
      {:else if value === 1}
        <NormalDistQ />
      {/if}
    </div>
  </div>
  <br /><br />
  <p class="body-text">And that's the end of our scrolly.</p>
</section>

<style>
  /* space after scroll is finished */
  .spacer {
    height: 40vh;
  }

  .charts-container {
    position: sticky;
    top: 10%;
    background: var(--bg);
    display: grid;
    width: 50%;
    grid-template-columns: 100%;
    grid-row-gap: 2rem;
    grid-column-gap: 0rem;
    grid-template-rows: repeat(2, 1fr);
    height: 85vh;
  }

  .section-container {
    margin-top: 1em;
    text-align: center;
    transition: background 100ms;
    display: flex;
  }

  .step {
    height: 110vh;
    display: flex;
    place-items: center;
    justify-content: center;
  }

  .step-content {
    font-size: 18px;
    background: var(--bg);
    color: var(--bg);
    border-radius: 1px;
    padding: 0.5rem 1rem;
    display: flex;
    flex-direction: column;
    justify-content: center;
    transition: background 500ms ease;
    text-align: left;
    width: 75%;
    margin: auto;
    max-width: 500px;
    font-family: var(--font-main);
    line-height: 1.3;
  }

  .step.active .step-content {
    background: #f7e8e4;
    color: var(--squid-ink);
    border: 2px solid var(--default);
  }

  .steps-container {
    height: 100%;
  }

  .steps-container {
    flex: 1 1 40%;
    z-index: 10;
  }

  /* Comment out the following line to always make it 'text-on-top' */
  @media screen and (max-width: 950px) {
    .section-container {
      flex-direction: column-reverse;
    }

    .steps-container {
      pointer-events: none;
    }

    .charts-container {
      top: 7.5%;
      width: 95%;
      margin: auto;
    }

    .step {
      height: 130vh;
    }

    .step-content {
      width: 95%;
      max-width: 768px;
      font-size: 17px;
      line-height: 1.6;
    }

    .spacer {
      height: 100vh;
    }
  }
</style>
