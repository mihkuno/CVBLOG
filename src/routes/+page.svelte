<script>
    import Math from '$lib/Math.svelte';
	import GradientMap from '$lib/GradientMap.svelte';
	import TrainingSimulator from '$lib/TrainingSimulator.svelte';
	import { fade, fly } from 'svelte/transition';

	// Svelte 5 State for the main page
	let batchSize = $state(32);

    let equation = "w_{t+1} = w_t - \\eta \\cdot \\frac{1}{B} \\sum_{i=1}^{B} \\nabla L(w_t, x_i, y_i)";

    // LaTeX strings for the list items
    const wNext = "w_{t+1}";
    const wCurrent = "w_t";
    const eta = "\\eta";
    const avgGrad = "\\frac{1}{B} \\sum_{i=1}^{B} \\nabla L(\\dots)";
    const lossFunc = "L";
    const batchsize = "B";
</script>

<div class="min-h-screen bg-slate-950 text-slate-200 selection:bg-blue-500 selection:text-white font-sans">
	
	<!-- Hero Section -->
	<header class="relative py-24 overflow-hidden border-b border-slate-900">
		<!-- Background Glow -->
		<div class="absolute inset-0 bg-[radial-gradient(ellipse_at_top,_var(--tw-gradient-stops))] from-indigo-900/20 via-slate-950 to-slate-950"></div>
		
		<div class="container mx-auto px-4 relative z-10 text-center max-w-4xl">
			<div class="inline-block mb-6 px-4 py-1.5 rounded-full bg-blue-900/20 border border-blue-500/20 text-blue-400 text-xs font-bold tracking-widest uppercase">
				CS Elective 1 - Graphics and Visual Computing
			</div>
			<h1 class="text-5xl md:text-7xl font-black text-transparent bg-clip-text bg-gradient-to-r from-blue-400 via-indigo-400 to-purple-400 mb-6 tracking-tighter leading-tight">
				The Batch Size Effect
			</h1>
			<p class="text-xl md:text-2xl text-slate-400 max-w-2xl mx-auto font-light leading-relaxed mb-8">
				Computer Vision Deep Learning Blog Post
			</p>
			
			<div class="flex items-center justify-center gap-4 text-slate-500">
				<div class="flex items-center gap-2 bg-slate-900/50 px-4 py-2 rounded-full border border-slate-800">
					<div class="w-2 h-2 rounded-full bg-green-500 animate-pulse"></div>
					<span class="font-medium text-slate-300">Joeniño D. Cainday</span>
				</div>
			</div>
		</div>
	</header>

	<main class="container mx-auto px-4 py-20 max-w-4xl">
		
		<!-- 1. Definition -->
		<section class="mb-24">
			<div class="flex items-baseline gap-4 mb-8">
				<span class="text-blue-500 font-mono text-sm">01</span>
				<h2 class="text-3xl md:text-4xl font-bold text-white">Definition & Purpose</h2>
			</div>
			
			<div class="grid md:grid-cols-3 gap-10 items-center">
				<div class="prose prose-invert prose-lg max-w-none text-slate-300 md:col-span-2">
					<p>
						In Deep Learning, we typically don't feed the model our entire dataset (which could be millions of images) at once. Instead, we break it into smaller chunks. The <strong>Batch Size</strong> is a hyperparameter that defines the number of training samples to work through before updating the internal model parameters.
					</p>
					<p>
						<strong>Purpose:</strong> Its primary role is to balance the computational efficiency of training (memory usage and speed) with the accuracy of the gradient descent optimization. It determines how often the model "learns" from its errors.
					</p>
				</div>

				<!-- Character Thinking about Batching -->
				<div class="md:col-span-1 flex justify-center ">
					<img src="/think.png" alt="Character thinking about batching" class="w-full max-w-xs" />
				</div>
			</div>
		</section>

        <!-- 2. Theoretical Effect (Enhanced & Intuitive) -->
<section class="mb-32">
	<div class="flex items-baseline gap-4 mb-10">
		<span class="text-blue-500 font-mono text-sm">02</span>
		<h2 class="text-3xl md:text-4xl font-bold text-white">Why Batch Size Changes Everything</h2>
	</div>

	<div class="prose prose-invert prose-lg max-w-none text-slate-300 space-y-8">

		<!-- Core Concept: The Loss Landscape (Updated Style) -->
		<div class="bg-slate-900/60 p-8 rounded-2xl border border-slate-800 shadow-xl">
			<h3 class="text-2xl font-bold text-white mb-6 flex items-center gap-3">
				<svg class="w-6 h-6 text-indigo-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 10h18M3 14h18m-9-4v8m-7 0h14a2 2 0 002-2V8a2 2 0 00-2-2H5a2 2 0 00-2 2v8a2 2 0 002 2z"></path></svg>
				Imagine the Loss Function as a Mountain Valley
			</h3>
			<p class="text-lg leading-relaxed">
				Training a neural network = trying to find the <strong>lowest point</strong> (minimum loss) in a vast, foggy, multidimensional valley.
			</p>
			<p class="mt-4 text-slate-400">
				You can’t see the whole valley. You only know: <em>“Which direction is downhill right now?”</em> → That’s your <strong>gradient</strong>.
			</p>
			<div class="my-8 flex justify-center">
				<div class="bg-slate-950/80 rounded-xl p-4 border border-blue-500/50 shadow-inner">
					<pre class="text-cyan-400 font-mono text-sm">∇L(w)  →  "Which way is downhill from where I stand?"</pre>
				</div>
			</div>
		</div>

		<!-- The Three Ways to Estimate the Gradient (Updated Borders/BGs) -->
		<div class="grid md:grid-cols-3 gap-6 my-12">
			<div class="bg-slate-900/50 backdrop-blur p-6 rounded-xl border border-emerald-600/50 hover:border-emerald-400 transition-all shadow-md">
				<div class="text-emerald-400 font-bold text-2xl mb-3">B = 1</div>
				<h4 class="text-white font-semibold mb-2">Stochastic (One Sample)</h4>
				<p class="text-sm text-slate-400">Ask <em>one random person</em>: “Which way down?”</p>
				<ul class="text-xs mt-3 space-y-1 text-slate-500">
					<li>Extremely noisy</li>
					<li>Fast updates</li>
					<li>Escapes sharp minima</li>
					<li>Like drunk walking</li>
				</ul>
			</div>

			<div class="bg-slate-900/70 backdrop-blur p-6 rounded-xl border border-blue-500/80 ring-2 ring-blue-500/30 shadow-2xl">
				<div class="text-blue-400 font-bold text-2xl mb-3">1 &lt; B &lt; N</div>
				<h4 class="text-white font-semibold mb-2">Mini-Batch (Sweet Spot)</h4>
				<p class="text-sm text-slate-400">Ask a <em>small group</em> and average their answers.</p>
				<ul class="text-xs mt-3 space-y-1 text-slate-500">
					<li>Balanced noise</li>
					<li>Efficient on GPU</li>
					<li>Best generalization</li>
					<li>Industry standard</li>
				</ul>
			</div>

			<div class="bg-slate-900/50 backdrop-blur p-6 rounded-xl border border-rose-600/50 hover:border-rose-400 transition-all shadow-md">
				<div class="text-rose-400 font-bold text-2xl mb-3">B = N</div>
				<h4 class="text-white font-semibold mb-2">Full Batch (All Data)</h4>
				<p class="text-sm text-slate-400">Survey <em>everyone</em> before deciding.</p>
				<ul class="text-xs mt-3 space-y-1 text-slate-500">
					<li>Very accurate gradient</li>
					<li>Slow (one update/epoch)</li>
					<li>Poor generalization</li>
					<li>Rarely used today</li>
				</ul>
			</div>
		</div>

		<!-- The Math (Updated Style) -->
		<div class="my-12 p-8 bg-slate-900/80 rounded-2xl border border-slate-700 shadow-xl">
			<h3 class="text-2xl font-bold text-white mb-4 flex items-center gap-3">
				<svg class="w-6 h-6 text-indigo-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 7h10m-5 5v5m0 0h-2m2 0h2m-2-5v5m0-5h-2m2 0h2m-2-5v5"></path></svg>
				The Gradient Descent Update Rule
			</h3>
			<p class="text-base text-slate-400 mb-6">
				This is the core equation for <strong>Mini-Batch Gradient Descent</strong>. It dictates how the model's weights (<Math tex="w" />) are adjusted at each step (<Math tex="t" />) to decrease the total loss. It's simply: 
				<strong class="text-white">New Weights = Current Weights - (Learning Rate * Average Gradient)</strong>.
			</p>
			
			<div class="grid md:grid-cols-2 gap-8 items-center">
				<div class="space-y-6">
					<div class="bg-slate-950 p-6 rounded-xl border border-blue-500/50 shadow-inner">
						<Math tex={equation}
						      class="text-2xl text-cyan-300" />
					</div>

                <ul class="space-y-3 text-sm text-slate-400">
                    <li class="flex items-center gap-2">
                        <strong class="text-white">The Goal: <Math tex={wNext} /></strong>: The new, improved set of model weights after the update.
                    </li>
                    <li class="flex items-center gap-2">
                        <strong class="text-white">The Starting Point: <Math tex={wCurrent} /></strong>: The current model weights.
                    </li>
                    <li class="flex items-center gap-2">
                        <strong class="text-white">The Step Size: <Math tex={eta} /></strong>: Learning rate — how far to jump in the downhill direction.
                    </li>
                    <li class="flex items-center gap-2">
                        <strong class="text-white">The Average Direction: <Math tex={avgGrad} /></strong>: The average gradient of the loss function <Math tex={lossFunc} /> over the <Math tex={batchSize} /> samples in the current batch.
                    </li>
                    <li class="flex items-center gap-2">
                        <strong class="text-white">The Batch Size: <Math tex={batchsize} /></strong>: The number of data points used to compute the average gradient, which is the core subject of our experiment.
                    </li>
                </ul>

				</div>

				<div class="bg-slate-900/50 p-6 rounded-xl border border-purple-500/50 shadow-inner">
					<h4 class="text-white font-semibold mb-4 text-2xl">Key Insight:</h4>
					<p class="text-lg leading-relaxed text-slate-300">
						The <strong>update direction</strong> is always the same formula.<br/>
						Only the <strong>quality of the gradient estimate</strong> changes with batch size.
					</p>
					<div class="mt-6 p-4 bg-slate-950/70 rounded-lg font-mono text-xs text-amber-300 border border-slate-700">
						Smaller B → Noisier ∇L → More exploration<br/>
						Larger B → Smoother ∇L → More exploitation
					</div>
				</div>
			</div>
		</div>

		<!-- Famous Quote + Research Insight -->
        <blockquote class="border-l-4 border-cyan-500 pl-6 py-4 my-12 italic text-lg text-cyan-300 bg-slate-900/50 rounded-r-xl">
			“Adding noise to the gradient is a feature, not a bug.”<br/>
			<span class="text-sm text-cyan-500 not-italic">— Keskar et al. (2016), Masters & Luschi (2018)</span>
		</blockquote>

		<p class="text-lg">
			Research has repeatedly shown: <strong>small batch training generalizes better</strong> than large batch training — 
			even when both reach the same training loss.
		</p>

		<div class="grid md:grid-cols-2 gap-8 my-12">
			<!-- Small Batches (Updated Style) -->
			<div class="bg-slate-900/50 border border-amber-500/50 p-6 rounded-xl shadow-md">
				<h4 class="text-amber-400 font-bold text-lg">Small Batches</h4>
				<p class="text-sm mt-2 text-slate-400">Find <strong>flat minima</strong> → robust to input shifts → better test accuracy</p>
			</div>
			<!-- Large Batches (Updated Style) -->
			<div class="bg-slate-900/50 border border-rose-500/50 p-6 rounded-xl shadow-md">
				<h4 class="text-rose-400 font-bold text-lg">Large Batches</h4>
				<p class="text-sm mt-2 text-slate-400">Converge to <strong>sharp minima</strong> → overfit → worse generalization</p>
			</div>
		</div>

		<!-- Interactive Visualization (No change needed here, it was already well-styled) -->
		<div class="mt-16 bg-slate-900/90 rounded-3xl border border-slate-800 p-10 shadow-2xl overflow-hidden">
			<div class="text-center mb-8">
				<h3 class="text-3xl font-bold text-white mb-3">
					See It In Action
				</h3>
				<p class="text-slate-400 max-w-2xl mx-auto">
					Drag the slider to control batch size. Watch how the optimization path changes in real time.
				</p>
			</div>

			<div class="grid lg:grid-cols-2 gap-10 items-center">
				<div class="space-y-8">
					<div class="bg-slate-950/80 p-8 rounded-2xl border border-slate-700">
						<label class="block text-lg font-medium text-slate-300 mb-6">
							Batch Size <span class="text-cyan-400 font-mono text-2xl ml-3">{batchSize}</span>
						</label>
						<input 
							type="range" 
							min="1" 
							max="128" 
							step="1"
							bind:value={batchSize}
							class="w-full h-4 bg-slate-800 rounded-full appearance-none cursor-pointer slider-thumb"
						/>
						<div class="flex justify-between text-xs text-slate-500 mt-3 font-mono">
							<span>Stochastic</span>
							<span>Mini-batch</span>
							<span class="text-right">Full batch →</span>
						</div>
					</div>

					<div class="text-sm space-y-3 text-slate-400">
						{#if batchSize <= 8}
							<div class="p-4 bg-fuchsia-900/30 border border-fuchsia-500/50 rounded-lg">
								<strong class="text-fuchsia-300">Very noisy path</strong> — explores widely, escapes local minima
							</div>
						{:else if batchSize <= 32}
							<div class="p-4 bg-blue-900/30 border border-blue-500/50 rounded-lg">
								<strong class="text-blue-300">Optimal zone</strong> — best generalization in practice
							</div>
						{:else}
							<div class="p-4 bg-emerald-900/30 border border-emerald-500/50 rounded-lg">
								<strong class="text-emerald-300">Smooth & direct</strong> — fast convergence, risk of overfitting
							</div>
						{/if}
					</div>
				</div>

				<div class="flex justify-center">
					<div class="w-full max-w-lg">
						<GradientMap {batchSize} />
					</div>
				</div>
			</div>

			<div class="mt-10 text-center text-sm text-slate-500 italic">
				Pink path = current batch size • Green path = large batch reference • Gray contours = loss landscape
			</div>
		</div>

		<!-- Final Takeaway (Updated Style) -->
		<div class="mt-16 p-8 bg-slate-900/60 rounded-2xl border border-blue-600/50 text-center shadow-xl">
			<h3 class="text-3xl font-bold text-white mb-4 flex items-center justify-center gap-3">
				<svg class="w-7 h-7 text-blue-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2m-9 0V3h10v2M9 5h6"></path></svg>
				Bottom Line
			</h3>
			<p class="text-lg max-w-4xl mx-auto leading-relaxed text-slate-300">
				Batch size is <strong>not just a speed knob</strong> — it’s a <strong>regularization parameter in disguise</strong>.<br/>
				Smaller batches inject healthy noise → better generalization.<br/>
				That’s why most winning models use batch sizes between 8 and 64 — never the full dataset.
			</p>
		</div>
	</div>
</section>

		<!-- 3. The Computational Trade-Off (NEW SECTION) -->
		<section class="mb-24">
			<div class="flex items-baseline gap-4 mb-8">
				<span class="text-blue-500 font-mono text-sm">03</span>
				<h2 class="text-3xl md:text-4xl font-bold text-white">The Computational Trade-Off</h2>
			</div>
			
			<div class="prose prose-invert prose-lg max-w-none text-slate-300">
				<p>
					While small batches often lead to better generalization (a lower validation loss), they come at a significant computational cost, creating a critical engineering trade-off for practitioners.
				</p>

				<h3 class="text-xl font-bold text-white mt-8 mb-4">Speed, Parallelism, and Memory</h3>
				<p>
					<strong>GPU Utilization:</strong> Modern GPUs are highly parallel devices. To maximize their computational throughput, you need to feed them enough data (a large batch) so that they are fully occupied. A batch size that is too small leaves compute units idle, resulting in inefficient training time per epoch.
				</p>
				<p>
					<strong>Memory Footprint:</strong> Conversely, the memory required to store the batch data, intermediate activations during the forward pass, and the gradients during the backward pass is directly proportional to the batch size. Exceeding the GPU's Video RAM (VRAM) capacity will cause the training run to crash.
				</p>
			</div>
			
			<!-- Visual Trade-Off Diagram -->
			<div class="mt-12 p-8 bg-slate-900 rounded-xl border border-slate-800 shadow-xl">
				<h3 class="text-xl font-bold text-white mb-6 text-center">Batch Size Trade-Off Visualizer</h3>
				<div class="flex flex-col items-center gap-2">
					
					<!-- Axis Labels -->
					<div class="flex justify-between w-full max-w-md px-1 text-xs text-slate-400 italic">
						<span>Small $B$</span>
						<span>Large $B$</span>
					</div>

					<!-- Slider/Ruler -->
					<div class="relative w-full max-w-md h-4 bg-gradient-to-r from-pink-700 via-slate-700 to-emerald-700 rounded-full">
						<div class="absolute inset-0 rounded-full border-2 border-slate-600"></div>
					</div>
					
					<!-- Arrows representing qualities -->
					<div class="grid grid-cols-2 gap-8 w-full max-w-md mt-6">
						
						<!-- LEFT SIDE: Small Batch Benefits -->
						<div class="flex flex-col items-start gap-3">
							<div class="flex items-center gap-2 text-pink-400 font-medium">
								<svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z"></path></svg>
								High Generalization
							</div>
							<div class="flex items-center gap-2 text-pink-400 font-medium">
								<svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8c-2.485 0-4.5 2.015-4.5 4.5s2.015 4.5 4.5 4.5 4.5-2.015 4.5-4.5-2.015-4.5-4.5-4.5zM12 1a1 1 0 011 1v1a1 1 0 01-2 0V2a1 1 0 011-1zM4 12a1 1 0 011-1h1a1 1 0 010 2H5a1 1 0 01-1-1zM18 12a1 1 0 011-1h1a1 1 0 010 2h-1a1 1 0 01-1-1zM12 18a1 1 0 011 1v1a1 1 0 01-2 0v-1a1 1 0 011-1zM6.586 5.586a1 1 0 011.414 0l.707.707a1 1 0 01-1.414 1.414l-.707-.707a1 1 0 010-1.414zM16.707 16.707a1 1 0 01-1.414 0l-.707-.707a1 1 0 011.414-1.414l.707.707a1 1 0 010 1.414z"></path></svg>
								Low Memory Usage
							</div>
						</div>

						<!-- RIGHT SIDE: Large Batch Benefits -->
						<div class="flex flex-col items-end gap-3 text-right">
							<div class="flex items-center gap-2 text-emerald-400 font-medium">
								High Training Stability
								<svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19V6l2-2l2 2v13"></path></svg>
							</div>
							<div class="flex items-center gap-2 text-emerald-400 font-medium">
								Fast Epoch Time
								<svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg>
							</div>
						</div>
					</div>
					
				</div>
			</div>
			
			<div class="prose prose-invert prose-lg max-w-none text-slate-300 mt-12">
				<h3 class="text-xl font-bold text-white mt-8 mb-4">Advanced Topic: Learning Rate Scaling</h3>
				<p>
					When training with extremely large batches (e.g., in distributed training), a common technique is to linearly increase the learning rate (<Math tex="\eta" />) proportionally to the batch size (<Math tex="B" />) to maintain the signal-to-noise ratio and ensure convergence speed. This is formalized by rules like the <strong>LARS (Layer-wise Adaptive Rate Scaling)</strong> or <strong>LPC (Linear Proportional Control)</strong> methods, which aim to mitigate the generalization gap caused by large batches.
				</p>
			</div>
		</section>

<!-- 4. Experimental Setup -->
<section class="mb-24">
	<div class="flex items-baseline gap-4 mb-8">
		<span class="text-blue-500 font-mono text-sm">04</span>
		<h2 class="text-3xl md:text-4xl font-bold text-white">Experimental Setup</h2>
	</div>

	<div class="grid md:grid-cols-2 gap-8">
		<!-- Dataset & Model -->
		<div class="bg-slate-900/50 backdrop-blur-sm p-8 rounded-xl border border-slate-800 shadow-xl">
			<h3 class="text-xl font-bold text-white mb-5 flex items-center gap-3">
				<svg class="w-6 h-6 text-blue-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
					<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19.428 15.428a2 2 0 00-1.022-.547l-2.384-.477a6 6 0 00-3.86.517l-.318.158a6 6 0 01-3.86.517L6.05 15.21a2 2 0 00-1.806.547M8 4h8l-1 1v5.172a2 2 0 00.586 1.414l5 5c1.26 1.26.367 3.414-1.415 3.414H4.828c-1.782 0-2.674-2.154-1.414-3.414l5-5A2 2 0 009 10.172V5L8 4z"></path>
				</svg>
				Dataset & Neural Network
			</h3>

			<div class="space-y-5 text-slate-300 text-sm leading-relaxed">
				<div>
					<h4 class="font-semibold text-white mb-2">Dataset: Fisher’s Iris (1936)</h4>
					<p>
						The classic <strong>Iris flower dataset</strong> contains <strong>150 samples</strong> from three species: 
						<em>Setosa</em>, <em>Versicolor</em>, and <em>Virginica</em> (50 each). 
						Each sample has 4 numerical features:
					</p>
					<ul class="mt-3 ml-6 list-disc space-y-1 text-slate-400">
						<li>Sepal Length (cm)</li>
						<li>Sepal Width (cm)</li>
						<li>Petal Length (cm)</li>
						<li>Petal Width (cm)</li>
					</ul>
					<p class="mt-3">
						<a href="https://gist.githubusercontent.com/netj/8836201/raw/6f9306ad21398ea43cba4f7d537619d0e07d5ae3/iris.csv" target="_blank" class="text-cyan-400 hover:text-cyan-300 underline font-medium">
							→ Full CSV on GitHub Gist
						</a>
					</p>
				</div>

				<div class="overflow-x-auto rounded-lg border border-slate-700">
					<table class="w-full text-xs font-mono">
						<thead>
							<tr class="bg-slate-800/50">
								<th class="text-left p-2 text-slate-400">sepal_length</th>
								<th class="text-left p-2 text-slate-400">sepal_width</th>
								<th class="text-left p-2 text-slate-400">petal_length</th>
								<th class="text-left p-2 text-slate-400">petal_width</th>
								<th class="text-left p-2 text-slate-400">species</th>
							</tr>
						</thead>
						<tbody class="divide-y divide-slate-700">
							<tr><td class="p-2">5.1</td><td class="p-2">3.5</td><td class="p-2">1.4</td><td class="p-2">0.2</td><td class="p-2 text-emerald-400">setosa</td></tr>
							<tr><td class="p-2">4.9</td><td class="p-2">3.0</td><td class="p-2">1.4</td><td class="p-2">0.2</td><td class="p-2 text-emerald-400">setosa</td></tr>
							<tr><td class="p-2">7.0</td><td class="p-2">3.2</td><td class="p-2">4.7</td><td class="p-2">1.4</td><td class="p-2 text-purple-400">versicolor</td></tr>
							<tr><td class="p-2">6.3</td><td class="p-2">3.3</td><td class="p-2">6.0</td><td class="p-2">2.5</td><td class="p-2 text-rose-400">virginica</td></tr>
							<tr><td class="p-2">5.8</td><td class="p-2">2.7</td><td class="p-2">5.1</td><td class="p-2">1.9</td><td class="p-2 text-rose-400">virginica</td></tr>
						</tbody>
					</table>
					<div class="text-center py-2 text-xs text-slate-500 bg-slate-800/30">
						… 145 more rows (150 total)
					</div>
				</div>

				<div>
					<h4 class="font-semibold text-white mb-2">Model Architecture</h4>
					<ul class="space-y-2 text-slate-400 text-sm">
						<li>• Input: 4 features → normalized to [0,1]</li>
						<li>• Hidden Layer: 32 units, ReLU activation</li>
						<li>• Output Layer: 3 units, Softmax (one-hot species)</li>
						<li>• Loss: Categorical Cross-Entropy</li>
						<li>• Optimizer: Stochastic Gradient Descent (learning rate = 0.1)</li>
						<li>• <strong>No pre-trained weights</strong> — trained from scratch in-browser</li>
					</ul>
				</div>
			</div>
		</div>

		<!-- Tools & Implementation -->
		<div class="bg-slate-900/50 backdrop-blur-sm p-8 rounded-xl border border-slate-800 shadow-xl">
			<h3 class="text-xl font-bold text-white mb-5 flex items-center gap-3">
				<svg class="w-6 h-6 text-emerald-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
					<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 20l4-16m4 4l4 4-4 4M6 16l-4-4 4-4"></path>
				</svg>
				Tools & Implementation
			</h3>

			<div class="space-y-6 text-slate-300 text-sm leading-relaxed">
				<div>
					<h4 class="font-semibold text-white mb-3">Core Libraries</h4>
					<div class="space-y-4">
						<div class="flex items-start gap-3">
							<div class="w-10 h-10 bg-blue-500/10 rounded-lg flex items-center justify-center flex-shrink-0">
								<span class="text-blue-400 font-bold text-lg">ml5</span>
							</div>
							<div>
								<strong class="text-white">ml5.js v1.3+</strong><br/>
								<span class="text-slate-400">
									High-level neural network API on top of TensorFlow.js. Handles data loading, normalization, training loop, and callbacks. Runs entirely in the browser.
								</span>
							</div>
						</div>

						<div class="flex items-start gap-3">
							<div class="w-10 h-10 bg-purple-500/10 rounded-lg flex items-center justify-center flex-shrink-0">
								<span class="text-purple-400 font-bold text-lg">p5</span>
							</div>
							<div>
								<strong class="text-white">p5.js + p5-svelte</strong><br/>
								<span class="text-slate-400">
									Real-time visualization of the training loss curve. Uses <code class="bg-slate-800 px-1 rounded">p5-svelte</code> for seamless Svelte integration.
								</span>
							</div>
						</div>

						<div class="flex items-start gap-3">
							<div class="w-10 h-10 bg-orange-500/10 rounded-lg flex items-center justify-center flex-shrink-0">
								<span class="text-orange-400 font-bold text-lg">TF</span>
							</div>
							<div>
								<strong class="text-white">TensorFlow.js (CPU backend)</strong><br/>
								<span class="text-slate-400">
									Forced to CPU backend via <code class="bg-slate-800 px-1 rounded">tf.setBackend('cpu')</code> and <code class="bg-slate-800 px-1 rounded">await tf.ready()</code> for maximum compatibility.
								</span>
							</div>
						</div>
					</div>
				</div>

				<div>
					<h4 class="font-semibold text-white mb-2">Key Implementation Details</h4>
					<ul class="space-y-2 text-slate-400 text-sm ml-1">
						<li>• Dynamic import of ml5.js only on client (SSR-safe)</li>
						<li>• Manual CSV → <code class="bg-slate-800 px-1 rounded">{`{xs, ys}`}</code> object parsing (required by ml5)</li>
						<li>• Real-time loss logging via <code class="bg-slate-800 px-1 rounded">whileTraining</code> callback</li>
						<li>• Interactive controls for batch size (1–64) and total epochs (10–200)</li>
						<li>• 20% validation split for live accuracy reporting</li>
					</ul>
				</div>

				<div class="pt-4 border-t border-slate-700">
					<p class="text-xs text-slate-500 italic">
						All training happens <strong>in the browser</strong> — no server, no GPU required. Perfect for demonstrating how batch size affects gradient noise and convergence speed.
					</p>
				</div>
			</div>
		</div>
	</div>
</section>

		<!-- 5. Simulation (Renumbered from 4) -->
		<section class="mb-24" id="simulation">
			<div class="flex items-baseline gap-4 mb-8">
				<span class="text-blue-500 font-mono text-sm">05</span>
				<h2 class="text-3xl md:text-4xl font-bold text-white">Live Experiment</h2>
			</div>
			
			<p class="text-slate-300 text-lg mb-8 max-w-2xl">
				Use the simulator below to vary the <strong>Batch Size</strong> and observe the real-time impact on the loss curve. Note the difference in curve smoothness and convergence speed.
			</p>
			
			<TrainingSimulator {batchSize} />
		</section>

		<!-- 6. Analysis (Renumbered from 5) -->
		<section class="mb-20">
			<div class="flex items-baseline gap-4 mb-8">
				<span class="text-blue-500 font-mono text-sm">06</span>
				<h2 class="text-3xl md:text-4xl font-bold text-white">Analysis & Findings</h2>
			</div>

			<div class="space-y-8">
				<!-- Question 1 -->
				<div class="bg-slate-900/30 border-l-2 border-blue-500 pl-6 py-2">
					<h3 class="text-lg font-bold text-white mb-2">What did you observe when changing the parameter?</h3>
					<p class="text-slate-300 leading-relaxed">
						When reducing the batch size (e.g., to 32 or 16), I observed that the loss curve became significantly more "jagged" or noisy. The model updates were frequent, causing the loss to fluctuate wildly. Conversely, increasing the batch size (e.g., to 128) resulted in a very smooth, monotonic decrease in loss, but each epoch took longer to process in a real-world scenario.
					</p>
				</div>

				<!-- Question 2 -->
				<div class="bg-slate-900/30 border-l-2 border-emerald-500 pl-6 py-2">
					<h3 class="text-lg font-bold text-white mb-2">Did it improve performance or stability?</h3>
					<p class="text-slate-300 leading-relaxed">
						It is a trade-off. Large batch sizes improved <strong>stability</strong> significantly; the gradient direction was consistent. However, small batch sizes often improved <strong>final performance</strong> (generalization). The instability seen in small batches actually prevented the model from overfitting to the training data, leading to higher validation accuracy in the long run.
					</p>
				</div>

				<!-- Question 3 -->
				<div class="bg-slate-900/30 border-l-2 border-purple-500 pl-6 py-2">
					<h3 class="text-lg font-bold text-white mb-2">What insights can you draw for future model tuning?</h3>
					<p class="text-slate-300 leading-relaxed">
						For future tuning, I will treat Batch Size not just as a speed parameter, but as a regularization hyperparameter. 
						<br/><br/>
						1. <strong>Start Moderate:</strong> Begin with 32 or 64. <br/>
						2. <strong>If Overfitting:</strong> Try reducing the batch size to introduce noise. <br/>
						3. <strong>If Unstable:</strong> If the loss diverges, increase the batch size or lower the learning rate.
					</p>
				</div>

				<!-- Data Table -->
				<div class="overflow-x-auto rounded-xl border border-slate-800 mt-10 shadow-xl">
					<table class="w-full text-left border-collapse">
						<thead>
							<tr class="bg-slate-900 text-slate-400 text-xs font-bold uppercase tracking-wider">
								<th class="p-5 border-b border-slate-800">Batch Size</th>
								<th class="p-5 border-b border-slate-800">Loss Curve Stability</th>
								<th class="p-5 border-b border-slate-800">Generalization</th>
								<th class="p-5 border-b border-slate-800">Memory Load</th>
							</tr>
						</thead>
						<tbody class="text-slate-300 divide-y divide-slate-800 bg-slate-900/20">
							<tr class="hover:bg-slate-800/50 transition-colors">
								<td class="p-5 font-mono text-pink-400">Small (8-32)</td>
								<td class="p-5">Low (Noisy)</td>
								<td class="p-5 text-emerald-400">High (Better)</td>
								<td class="p-5 text-emerald-400">Low</td>
							</tr>
							<tr class="hover:bg-slate-800/50 transition-colors">
								<td class="p-5 font-mono text-emerald-400">Large (128+)</td>
								<td class="p-5 text-emerald-400">High (Smooth)</td>
								<td class="p-5">Moderate</td>
								<td class="p-5 text-pink-400">High</td>
							</tr>
						</tbody>
					</table>
				</div>
			</div>
			
			<div class="mt-12 pt-8 border-t border-slate-800 text-slate-500 text-sm">
				<h4 class="text-slate-400 font-bold uppercase mb-2">References</h4>
				<ul class="list-disc pl-5 space-y-1">
					<li>Keskar, N. S., et al. (2016). <em>On Large-Batch Training for Deep Learning: Generalization Gap and Sharp Minima</em>. ICLR.</li>
					<li>Masters, D., & Luschi, C. (2018). <em>Revisiting Small Batch Training for Deep Neural Networks</em>. arXiv.</li>
					<li>Goodfellow, I., Bengio, Y., & Courville, A. (2016). <em>Deep Learning</em>. MIT Press.</li>
				</ul>
			</div>
		</section>

	</main>

	<footer class="border-t border-slate-900 py-12 bg-slate-950 text-center">
		<p class="text-slate-600 text-sm">
			© 2025 Joeniño D. Cainday. <br/>
			CS Elective 1 - Graphics and Visual Computing.
		</p>
	</footer>
</div>

<style>
    	.slider-thumb::-webkit-slider-thumb {
		appearance: none;
		width: 28px;
		height: 28px;
		background: #06b6d4;
		border-radius: 50%;
		box-shadow: 0 0 20px rgba(6, 182, 212, 0.6);
		cursor: grab;
	}
	.slider-thumb::-moz-range-thumb {
		width: 28px;
		height: 28px;
		background: #06b6d4;
		border-radius: 50%;
		box-shadow: 0 0 20px rgba(6, 182, 212, 0.6);
		cursor: grab;
		border: none;
	}
</style>