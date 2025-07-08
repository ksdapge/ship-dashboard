<script>
  let height = '';
  let heightUnit = 'feet';
  let temperature = '';
  let tempUnit = 'F';
  let tempWarning = '';
  let imgType = '';
  let load = null;

  function convertHeightToFeet(h, unit) {
    return unit === 'meters' ? h * 3.2808399 : h;
  }

  function convertTempToFahrenheit(t, unit) {
    return unit === 'C' ? (t * 9/5) + 32 : t;
  }

  function getLoad(tempF, airGapFt) {
    const lookupTable = {
      "50-79": [
        [129.942, 350],
        [130.892, 300],
        [131.642, 250],
        [132.192, 200],
        [132.602, 150],
        [132.892, 100],
        [133.052, 50],
        [133.102, 0]
      ],
      "80-89": [
        [129.172, 350],
        [130.152, 300],
        [130.912, 250],
        [131.492, 200],
        [131.922, 150],
        [132.202, 100],
        [132.382, 50],
        [132.432, 0]
      ],
      "90-99": [
        [128.412, 350],
        [129.422, 300],
        [130.212, 250],
        [130.802, 200],
        [131.242, 150],
        [131.542, 100],
        [131.712, 50],
        [131.772, 0]
      ],
      "100-109": [
        [127.662, 350],
        [128.702, 300],
        [129.502, 250],
        [130.112, 200],
        [130.572, 150],
        [130.872, 100],
        [131.062, 50],
        [131.122, 0]
      ],
      "110+": [
        [126.962, 350],
        [127.982, 300],
        [128.812, 250],
        [129.442, 200],
        [129.902, 150],
        [130.222, 100],
        [130.412, 50],
        [130.472, 0]
      ],
      "tooHigh": [
        [0, 0]
      ]
    };

    let rangeKey = null;

    if (tempF >= 50 && tempF <= 79) rangeKey = "50-79";
    else if (tempF >= 80 && tempF < 90) rangeKey = "80-89";
    else if (tempF >= 90 && tempF < 100) rangeKey = "90-99";
    else if (tempF >= 100 && tempF < 110) rangeKey = "100-109";
    else if (tempF >= 110 && tempF < 120) rangeKey = "110+";
    else if (tempF >= 120) rangeKey = "tooHigh";

    const thresholds = lookupTable[rangeKey];

    if (airGapFt < 126) {return 350;}
    if (rangeKey === 'tooHigh') {return 0;} 
    else {
      for (let [threshold, load] of thresholds) {
        if (airGapFt <= threshold) {
          return load;
        } 
      }
      return 0;
    }
  }

  function calculateLoad() {
    load = null;

    const h = parseFloat(height);
    const t = parseFloat(temperature);

    if (isNaN(h) || isNaN(t)) {
      load = 'Invalid input';
      return;
    }

    const hFeet = convertHeightToFeet(h, heightUnit);
    const tFah = convertTempToFahrenheit(t, tempUnit);
    console.log(`Height in feet: ${hFeet}, Temperature in Fahrenheit: ${tFah}`);

    if (hFeet > 133.105) {
      tempWarning = 'SHIP IS TOO TALL TO PASS, LINE MUST BE CUT';
      imgType = 'stop';
      load = '0 A';
      return;
    }

    const result = getLoad(tFah, hFeet);
    console.log(`Calculated load: ${result}`);
    load = typeof result === 'number' ? `${result} A` : result;

    if (tFah < 50) {
      tempWarning = 'Temperature is low...are you sure this is the correct projected temperature?';
      imgType = 'boat';
    } else if (tFah > 119) {
      tempWarning = 'VERY HIGH TEMPERATURE. NEED TO RE-EVALUATE BASED ON EXACT TEMPERATURE.';
      imgType = 'stop';
    } else if (result === 0) {
      tempWarning = 'MUST DROP ALL LOAD';
      imgType = 'stop';
    } else {
      imgType = 'boat';
    }
  }
</script>

<main>
  <h1 style="color:#0089c4">Max Load For Ship Crossing Calculator</h1>

  <div>
    <label>Ship Height:</label>
    <input type="number" bind:value={height} />
    <select bind:value={heightUnit}>
      <option value="feet">Feet</option>
      <option value="meters">Meters</option>
    </select>
  </div>

  <div>
    <label>Projected Temperature:</label>
    <input type="number" bind:value={temperature} />
    <select bind:value={tempUnit}>
      <option value="F">Fahrenheit</option>
      <option value="C">Celsius</option>
    </select>
  </div>

  <div style="margin-top: 1rem;">
    <button on:click={calculateLoad}>Calculate Load</button>
  </div>

  {#if load !== null}
    <h2><strong>Max Load Capability:</strong> {load}</h2>
  {/if}

  <div class="image-container">
    {#if tempWarning}
      <p style="color:red"><strong>CAUTION:</strong> {tempWarning}</p>
    {/if}

    {#if imgType === 'stop'}
      <img src="https://i.pinimg.com/736x/8c/7c/01/8c7c01345fbb38d6bf4f317c48899926.jpg" />
    {:else if imgType === 'boat'}
      <img src="https://content.presentermedia.com/files/clipart/00002000/2992/stick_figure_thumbs_up_pc_800_wht.jpg" />
    {/if}
  </div>
</main>

<style>
  :global(body) {
    background-color: white;
    color: black;
    margin: 0;
    font-family: arial;
  }

  main {
    font-family: arial;
    padding: 2rem;
    max-width: 1080px;
    margin: auto;
  }

  div {
    margin-bottom: 1rem;
  }

  input, select, button {
    margin-left: 0.5rem;
    color: black;
    background-color: lightgray;
  }
</style>
