<script>
  import P5 from 'p5-svelte';
  import { onMount } from 'svelte';
  import { browser } from '$app/environment';

  // Reactive state
  let batchSize = 16;
  let epochs = 50;
  let isTraining = false;
  let currentEpoch = 0;
  let currentLoss = 0;
  let accuracy = 0;
  let lossHistory = [];

  // ml5 and tf loaded only in browser
  let ml5 = null;
  let tf = null;
  let nn = null;

  // p5 sketch for loss curve (unchanged from previous)
  const sketch = (p) => {
    let maxLoss = 2.5;

    p.setup = () => {
      p.createCanvas(600, 280);
      p.textFont('monospace', 12);
      p.textAlign(p.LEFT, p.TOP);
    };

    p.draw = () => {
      p.background('#0f172a');

      if (lossHistory.length === 0) {
        p.fill('#64748b');
        p.noStroke();
        p.textSize(18);
        p.text('Click "Train Model" to see the loss curve', 20, p.height / 2);
        return;
      }

      // Dynamic max loss
      maxLoss = Math.max(2.5, ...lossHistory) * 1.1;

      // Axes
      p.stroke('#334155');
      p.strokeWeight(2);
      const margin = 40;
      p.line(margin, margin, p.width - margin, margin); // top
      p.line(margin, margin, margin, p.height - margin); // left
      p.line(margin, p.height - margin, p.width - margin, p.height - margin); // bottom
      p.line(p.width - margin, margin, p.width - margin, p.height - margin); // right

      // Labels
      p.fill('#94a3b8');
      p.textSize(12);
      p.text('Loss', margin - 30, margin / 2);
      p.rotate(-Math.PI / 2);
      p.text('Epochs →', -(p.height / 2), margin - 20);
      p.rotate(Math.PI / 2);

      // Loss curve
      p.stroke('#38bdf8');
      p.strokeWeight(3);
      p.noFill();
      p.beginShape();
      for (let i = 0; i < lossHistory.length; i++) {
        const x = p.map(i, 0, epochs - 1, margin, p.width - margin);
        const y = p.map(lossHistory[i], 0, maxLoss, p.height - margin, margin);
        p.vertex(x, y);
      }
      p.endShape();

      // Current point highlight
      if (lossHistory.length > 0) {
        const i = lossHistory.length - 1;
        const x = p.map(i, 0, epochs - 1, margin, p.width - margin);
        const y = p.map(lossHistory[i], 0, maxLoss, p.height - margin, margin);
        p.fill('#38bdf8');
        p.noStroke();
        p.circle(x, y, 8);
        p.fill('#0f172a');
        p.circle(x, y, 4);
      }
    };
  };

  onMount(async () => {
    if (!browser) return;
    const ml5Module = await import('ml5');
    ml5 = ml5Module.default || ml5Module;

    // Access tfjs from ml5
    tf = ml5.tf; // tfjs is exposed via ml5.tf

    // Ensure tf is ready and force CPU backend to avoid WebGPU issues
    await tf.setBackend('cpu');
    await tf.ready();
    console.log('TensorFlow.js ready on CPU backend');
  });

  // Helper: Parse raw CSV rows to ml5 object format (unchanged)
  function parseIrisToMl5Format(rows) {
    const [header, ...dataRows] = rows;
    const inputCols = header.slice(0, -1); // Exclude 'species'
    const samples = [];

    dataRows.forEach(row => {
      if (row.length < 5) return; // Skip invalid rows
      const xs = {};
      inputCols.forEach((col, i) => {
        xs[col] = parseFloat(row[i]) || 0; // Convert to numbers
      });
      const ys = { species: row[4] }; // String label
      samples.push({ xs, ys });
    });

    console.log(`Parsed ${samples.length} Iris samples`);
    return samples;
  }

  async function startTraining() {
    if (!browser || !ml5 || !tf || isTraining) return;

    try {
      // Double-check tf is ready (safety net)
      if (!tf.backend() || tf.backend() !== 'cpu') {
        await tf.setBackend('cpu');
        await tf.ready();
      }

      // Reset state
      isTraining = true;
      currentEpoch = 0;
      currentLoss = 0;
      accuracy = 0;
      lossHistory = [];

      // Robust network options
      const options = {
        inputs: ['sepal_length', 'sepal_width', 'petal_length', 'petal_width'],
        outputs: ['species'],
        task: 'classification',
        debug: false,
        layers: [
          { type: 'dense', units: 32, activation: 'relu' },
          { type: 'dense', units: 3, activation: 'softmax' }
        ],
        learningRate: 0.1
      };

      nn = ml5.neuralNetwork(options);

      // Load raw CSV as text
      const response = await fetch(
        'https://gist.githubusercontent.com/netj/8836201/raw/6f9306ad21398ea43cba4f7d537619d0e07d5ae3/iris.csv'
      );
      const csvText = await response.text();

      // Parse to ml5 format
      const rows = csvText.split('\n').map(row => row.split(','));
      const parsedData = parseIrisToMl5Format(rows);

      if (parsedData.length === 0) {
        throw new Error('Failed to parse Iris data');
      }

      // Add each sample individually
      parsedData.forEach(sample => {
        nn.addData(sample.xs, sample.ys);
      });

      // Normalize data
      nn.normalizeData();

      const trainOpts = {
        epochs,
        batchSize,
        validationSplit: 0.2
      };

      nn.train(
        trainOpts,
        (epoch, logs) => {
          currentEpoch = epoch + 1;
          currentLoss = logs.loss?.toFixed(4) || 0;
          accuracy = (logs.acc ? logs.acc * 100 : 0).toFixed(1);
          lossHistory = [...lossHistory, logs.loss || 0];
          console.log(`Epoch ${epoch}: Loss=${currentLoss}, Acc=${accuracy}%`);
        },
        () => {
          console.log('Training complete!');
          isTraining = false;
        }
      );
    } catch (error) {
      console.error('Training error:', error);
      isTraining = false;
      alert(`Training failed: ${error.message}`);
    }
  }

  function reset() {
    if (nn) nn = null;
    lossHistory = [];
    currentEpoch = 0;
    currentLoss = 0;
    accuracy = 0;
    isTraining = false;
  }
</script>

{#if browser}
  <div class="max-w-6xl mx-auto p-8 font-sans text-white bg-slate-900 min-h-screen">
    <h1 class="text-4xl font-bold mb-2 text-center">ml5.js Iris Training Demo</h1>
    <p class="text-slate-400 mb-8 text-center">Watch batch size affect the loss curve on real Iris data!</p>

    <div class="grid lg:grid-cols-2 gap-8">
      <!-- Controls -->
      <div class="space-y-6">
        <div class="bg-slate-800/80 backdrop-blur rounded-xl p-6 border border-slate-700">
          <h2 class="text-xl font-semibold mb-6">Parameters</h2>
          <div class="space-y-6">
            <div>
              <label class="block text-slate-300 mb-2">
                Batch Size: <span class="text-cyan-400 font-bold">{batchSize}</span>
              </label>
              <input
                type="range"
                min="1"
                max="64"
                bind:value={batchSize}
                disabled={isTraining}
                class="w-full h-3 bg-slate-700 rounded-lg slider"
              />
              <div class="flex justify-between text-xs text-slate-500 mt-2">
                <span>1 (Noisy)</span><span>32</span><span>64 (Smooth)</span>
              </div>
            </div>

            <div>
              <label class="block text-slate-300 mb-2">
                Epochs: <span class="text-purple-400 font-bold">{epochs}</span>
              </label>
              <input
                type="range"
                min="10"
                max="200"
                step="10"
                bind:value={epochs}
                disabled={isTraining}
                class="w-full h-3 bg-slate-700 rounded-lg slider"
              />
            </div>

            <div class="flex gap-4">
              <button
                on:click={startTraining}
                disabled={isTraining || !ml5 || !tf}
                class="flex-1 py-4 bg-gradient-to-r from-cyan-500 to-blue-600 hover:from-cyan-400 hover:to-blue-500 disabled:opacity-50 font-bold rounded-lg shadow-lg transition transform hover:scale-105"
              >
                {isTraining ? `Training... (${currentEpoch}/${epochs})` : '🚀 Train Model'}
              </button>
              <button
                on:click={reset}
                disabled={isTraining}
                class="px-6 py-4 bg-slate-700 hover:bg-slate-600 disabled:opacity-50 rounded-lg transition"
              >
                Reset
              </button>
            </div>
          </div>
        </div>

        <!-- Live Stats -->
        <div class="grid grid-cols-3 gap-4">
          <div class="bg-slate-900/80 backdrop-blur p-5 rounded-lg border border-slate-700 text-center">
            <div class="text-slate-400 text-sm uppercase tracking-wider">Epoch</div>
            <div class="text-3xl font-mono text-white">{currentEpoch}/{epochs}</div>
          </div>
          <div class="bg-slate-900/80 backdrop-blur p-5 rounded-lg border border-slate-700 text-center">
            <div class="text-slate-400 text-sm uppercase tracking-wider">Loss</div>
            <div class="text-3xl font-mono text-rose-400">{currentLoss || '—'}</div>
          </div>
          <div class="bg-slate-900/80 backdrop-blur p-5 rounded-lg border border-slate-700 text-center">
            <div class="text-slate-400 text-sm uppercase tracking-wider">Accuracy</div>
            <div class="text-3xl font-mono text-emerald-400">{accuracy}%</div>
          </div>
        </div>
      </div>

      <!-- Loss Chart -->
      <div class="bg-slate-900/80 backdrop-blur rounded-xl border border-slate-700 overflow-hidden shadow-2xl">
        <div class="p-4 border-b border-slate-700 bg-slate-800/50">
          <h3 class="text-lg font-semibold">Loss Over Epochs</h3>
          <p class="text-sm text-slate-500">Blue curve shows training progress</p>
        </div>
        <div class="p-2">
          <P5 {sketch} />
        </div>
      </div>
    </div>

    <div class="mt-8 text-center text-sm text-slate-500">
      <strong>Pro Tip:</strong> Small batch size = zig-zag (noisy gradients, faster exploration) | Large = smooth (stable but slower)
    </div>
  </div>
{:else}
  <div class="flex items-center justify-center min-h-screen text-slate-400">
    Loading ML demo (browser only)...
  </div>
{/if}

<style>
  :global(body) {
    margin: 0;
    background: #0f172a;
  }
  .slider::-webkit-slider-thumb {
    appearance: none;
    height: 20px;
    width: 20px;
    border-radius: 50%;
    background: #38bdf8;
    cursor: pointer;
    box-shadow: 0 0 10px rgba(56, 189, 248, 0.5);
  }
  .slider::-moz-range-thumb {
    height: 20px;
    width: 20px;
    border-radius: 50%;
    background: #38bdf8;
    cursor: pointer;
    border: none;
  }
</style>