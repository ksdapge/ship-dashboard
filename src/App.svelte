<script>
  document.title = "Ship Crossing Calculator"

  import { fade, scale } from 'svelte/transition';

  let heightFeet = '';
  let heightMeters = '';
  let temperature = '';
  let tempUnit = 'F';
  let tempWarning = '';
  let imgType = '';
  let load = null;
  let showPopup = false;
  let popupMessage = '';


  function feetToMeters(feet) {
    return feet / 3.2808399;
  }

  function metersToFeet(meters) {
    return meters * 3.2808399;
  }

  function onFeetInput(e) {
    heightFeet = e.target.value;
    const feet = parseFloat(heightFeet);
    if (!isNaN(feet)) {
      heightMeters = (feetToMeters(feet)).toFixed(2);
    } else {
      heightMeters = '';
    }
  }

  function onMetersInput(e) {
    heightMeters = e.target.value;
    const meters = parseFloat(heightMeters);
    if (!isNaN(meters)) {
      heightFeet = (metersToFeet(meters)).toFixed(2);
    } else {
      heightFeet = '';
    }
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

    if (airGapFt < 126) { return 350; }
    if (rangeKey === 'tooHigh') { return 0; }
    else {
      for (let [threshold, load] of thresholds) {
        if (airGapFt <= threshold) {
          return load;
        }
      }
      return 0;
    }
  }

  const okImages = [
    'images/np1.jpg',
    'images/np2.jpg',
    'images/np3.jpg'
  ];

  const cautionImages = [
    'images/caution.jpg',
    'images/caution1.jpg',
    'images/caution2.jpg'
  ];

  function pickRandom(arr) {
    return arr[Math.floor(Math.random() * arr.length)];
  }
  
  function getMaxAirGapForTemp(tempF) {
  if (tempF >= 50 && tempF <= 79) return 133.102;
  if (tempF >= 80 && tempF < 90) return 132.432;
  if (tempF >= 90 && tempF < 100) return 131.772;
  if (tempF >= 100 && tempF < 110) return 131.122;
  if (tempF >= 110 && tempF < 120) return 130.472;
  return null;
  }
  
  function calculateLoad() {
    tempWarning = '';
    load = null;
    showPopup = false;
    popupMessage = '';

    const hFeet = parseFloat(heightFeet);
    const t = parseFloat(temperature);

    if (isNaN(hFeet) || isNaN(t)) {
      load = 'Invalid input';
      return;
    }

    const tFah = convertTempToFahrenheit(t, tempUnit);

    const maxAirGap = getMaxAirGapForTemp(tFah);
    if (maxAirGap !== null && hFeet > maxAirGap) {
      tempWarning = 'SHIP CANNOT PASS, IT IS TOO TALL!!!';
      imgType = 'images/stop.jpg';
      load = 'CAUTION!';
      showPopup = false;
      return;
    }

    if (hFeet > 133.105) {
      tempWarning = 'SHIP CANNOT PASS, IT IS TOO TALL!!!';
      imgType = 'images/stop.jpg';
      load = 'CAUTION!';
      showPopup = false;
      return;
    }

    const result = getLoad(tFah, hFeet);
    load = `${result} A`;

    // Image + warning logic
    if (tFah < 50) {
      tempWarning = 'Temperature is low...are you sure this is the correct projected temperature?';
      imgType = pickRandom(cautionImages);
    }
    else if (tFah > 119) {
      tempWarning = 'VERY HIGH TEMPERATURE. NEED TO RE-EVALUATE BASED ON EXACT TEMPERATURE.';
      imgType = pickRandom(cautionImages);
    }
    else if ([250, 200, 150, 50, 0].includes(result)) {
      tempWarning = result === 0 ? 'MUST DROP ALL LOAD FOR SHIP TO PASS' : '';
      imgType = pickRandom(cautionImages);
    }
    else {
      imgType = pickRandom(okImages);
    }
    
    
    
    if (
      ![350, 300].includes(result) &&
      tempWarning !== 'SHIP CANNOT PASS, IT IS TOO TALL!!!' &&
      tempWarning !== 'VERY HIGH TEMPERATURE. NEED TO RE-EVALUATE BASED ON EXACT TEMPERATURE.'
    ) {
      popupMessage = 'CAUTION: Max load capability is reduced.';
      showPopup = true;
    }


  }

</script>

<main>
  <h1 style="color:#0089c4">Max Load For Ship Crossing Calculator</h1>

  <div>
    <label style="color:#fbbb36"><strong>Ship Height (Fill Out Either Box):</strong></label>
    <div class="height-inputs">
      <div class="input-group">
        <label for="feet">Feet:</label>
        <input id="feet" type="number" bind:value={heightFeet} on:input={onFeetInput} />
      </div>
      <div class="input-group">
        <label for="meters">Meters:</label>
        <input id="meters" type="number" bind:value={heightMeters} on:input={onMetersInput} />
      </div>
    </div>
  </div>

  
<div>
  <!-- Line 1: Label -->
  <label style="color:#fbbb36"><strong>Projected Temperature:</strong></label>

  <!-- Line 2: Hyperlink -->
  <a href="https://www.accuweather.com/en/us/rio-vista/94571/daily-weather-forecast/2154362" target="_blank" style="color:#0089c4; text-decoration: underline;">
    AccuWeather Rio Vista
  </a>


  <!-- Line 3: Input and Select -->
  <div>
    <input type="number" bind:value={temperature} />
    <select bind:value={tempUnit}>
    <option value="F">Fahrenheit</option>
    <option value="C">Celsius</option>
    </select>
  </div>
</div>

<div style="margin-top: 1rem;">
  <button on:click={calculateLoad}>
    Calculate Load
  </button>
</div>

<style>
  button {
    background-color: #4f46e5; /* normal color */
    color: white;
    padding: 0.5rem 1rem;
    border: none;
    border-radius: 6px;
    cursor: pointer;
  }

  button:active {
    background-color: #95bd8a; /* pressed color */
  }
</style>


{#if load !== null}
  <h2><strong>Max Load Capability (on line):</strong> {load}</h2>
{/if}

<div class="image-container">
  {#if tempWarning}
    <p style="color:red"><strong>CAUTION:</strong> {tempWarning}</p>
  {/if}

  {#if imgType}
    <img src={imgType} alt="status image" />
  {/if}

</div>


{#if showPopup}
  <div class="modal-backdrop" transition:fade={{ duration: 300 }}>
    <div class="modal">
      <div class="popup-warning">
        <p class="popup-title">
          CAUTION: Max Load Capability is Reduced
        </p>

        <p>Take the following steps:</p>

        <ol>
          <li>
            Verify in CYME the correct loading on the line
            based on the nearest upstream metering or SCADA device.
          </li>
          <li>
            If projected load exceeds calculated load capability,
            develop a load transfer plan to get the desired loading.
          </li>
          <li>
            Once the plan is developed, send email with your suggested plan to:
            <br />
            <strong>
              CentralDCCSupervisors@pge.com,<br />
              P1O2@pge.com,<br />
              P2B5@pge.com,<br />
              MXBQ@pge.com <br />

              and cc PGEShipNotification@pge.com
            </strong>
            <br /><br />
            and <strong>MARK HIGH IMPORTANCE</strong> with the following text:
            <br /><br />
            <em>
              “We will need to perform a load transfer to allow this ship to pass
              on [insert date]. Attached is my suggested SW plan to transfer
              [amps] to [circuit]. I will work with LSOs to submit an AFW before
              the ship passes.”
            </em>
          </li>
          <li>
            Work with NB LSO to submit an AFW to execute the
            suggested load transfer.
          </li>
        </ol>
        
        <button on:click={() => (showPopup = false)}>OK</button>
      </div>
    </div>
  </div>
{/if}


<br>

<div>
  <a 
    href="https://edpi.cloud.pge.com/edpi/trend?did={ECF9A66B-568D-4B4A-82B5-3C447A75F852}" 
    target="_blank" 
    rel="noopener noreferrer"
    style="color:#0089c4; text-decoration: underline; font-weight: bold;"
  >
    EDPI Link
  </a>
</div>

<h2>Ship Load Lookup Table</h2>

<h3 style="color:#a30505; font-size:1rem;">
  If 120°F or greater, must evaluate on case-by-case basis...
</h3>


<div class="table-grid">

  <!-- 70–79 -->
  <table class="temp-table">
    <thead>
      <tr><th colspan="2">70–79 °F</th></tr>
      <tr><th>Air Gap</th><th>Load</th></tr>
    </thead>
    <tbody>
      <tr style="background:#d0e7ff"><td>129.942</td><td>350.0</td></tr>
      <tr><td>130.892</td><td>300.0</td></tr>
      <tr style="background:#d0e7ff"><td>131.642</td><td>250.0</td></tr>
      <tr><td>132.192</td><td>200.0</td></tr>
      <tr style="background:#d0e7ff"><td>132.602</td><td>150.0</td></tr>
      <tr><td>132.892</td><td>100.0</td></tr>
      <tr style="background:#d0e7ff"><td>133.052</td><td>50.0</td></tr>
      <tr><td>133.102</td><td>0.0</td></tr>
    </tbody>
  </table>

  <!-- 80–89 -->
  <table class="temp-table">
    <thead>
      <tr><th colspan="2">80–89 °F</th></tr>
      <tr><th>Air Gap</th><th>Load</th></tr>
    </thead>
    <tbody>
      <tr style="background:#d6f5d6"><td>129.172</td><td>350.0</td></tr>
      <tr><td>130.152</td><td>300.0</td></tr>
      <tr style="background:#d6f5d6"><td>130.912</td><td>250.0</td></tr>
      <tr><td>131.492</td><td>200.0</td></tr>
      <tr style="background:#d6f5d6"><td>131.922</td><td>150.0</td></tr>
      <tr><td>132.202</td><td>100.0</td></tr>
      <tr style="background:#d6f5d6"><td>132.382</td><td>50.0</td></tr>
      <tr><td>132.432</td><td>0.0</td></tr>
    </tbody>
  </table>

  <!-- 90–99 -->
  <table class="temp-table">
    <thead>
      <tr><th colspan="2">90–99 °F</th></tr>
      <tr><th>Air Gap</th><th>Load</th></tr>
    </thead>
    <tbody>
      <tr style="background:#fff9cc"><td>128.412</td><td>350.0</td></tr>
      <tr><td>129.422</td><td>300.0</td></tr>
      <tr style="background:#fff9cc"><td>130.212</td><td>250.0</td></tr>
      <tr><td>130.802</td><td>200.0</td></tr>
      <tr style="background:#fff9cc"><td>131.242</td><td>150.0</td></tr>
      <tr><td>131.542</td><td>100.0</td></tr>
      <tr style="background:#fff9cc"><td>131.712</td><td>50.0</td></tr>
      <tr><td>131.772</td><td>0.0</td></tr>
    </tbody>
  </table>

  <!-- 100–109 -->
  <table class="temp-table">
    <thead>
      <tr><th colspan="2">100–109 °F</th></tr>
      <tr><th>Air Gap</th><th>Load</th></tr>
    </thead>
    <tbody>
      <tr style="background:#ffe0b3"><td>127.662</td><td>350.0</td></tr>
      <tr><td>128.702</td><td>300.0</td></tr>
      <tr style="background:#ffe0b3"><td>129.502</td><td>250.0</td></tr>
      <tr><td>130.112</td><td>200.0</td></tr>
      <tr style="background:#ffe0b3"><td>130.572</td><td>150.0</td></tr>
      <tr><td>130.872</td><td>100.0</td></tr>
      <tr style="background:#ffe0b3"><td>131.062</td><td>50.0</td></tr>
      <tr><td>131.122</td><td>0.0</td></tr>
    </tbody>
  </table>

  <!-- 110–119 -->
  <table class="temp-table">
    <thead>
      <tr><th colspan="2">110–119 °F</th></tr>
      <tr><th>Air Gap</th><th>Load</th></tr>
    </thead>
    <tbody>
      <tr style="background:#ffd6d6"><td>126.962</td><td>350.0</td></tr>
      <tr><td>127.982</td><td>300.0</td></tr>
      <tr style="background:#ffd6d6"><td>128.812</td><td>250.0</td></tr>
      <tr><td>129.442</td><td>200.0</td></tr>
      <tr style="background:#ffd6d6"><td>129.902</td><td>150.0</td></tr>
      <tr><td>130.222</td><td>100.0</td></tr>
      <tr style="background:#ffd6d6"><td>130.412</td><td>50.0</td></tr>
      <tr><td>130.472</td><td>0.0</td></tr>
    </tbody>
  </table>

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

  .image-container img {
    max-width: 300px;
    margin-top: 1rem;
  }

  .height-inputs {
    display: flex;
    justify-content: center;
    gap: 2rem;
    margin-top: 0.5rem;
  }

  .input-group {
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .input-group label {
    min-width: 50px;
    text-align: right;
    font-weight: bold;
  }
</style>
