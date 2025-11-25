<script>
	import { onMount } from 'svelte';

	let { batchSize = 32, learningRate = 0.05 } = $props();

	let container;
	let myP5;
	let path = $state([]);
	let position = { x: 50, y: 50 }; // Starting top-left

	$effect(() => {
		// Reset path when batch size changes drastically to show the contrast
		if (batchSize) restart();
	});

	function restart() {
		position = { x: 50, y: 50 };
		path = [];
	}

	onMount(async () => {
		const p5 = (await import('p5')).default;

		const sketch = (p) => {
			p.setup = () => {
				p.createCanvas(400, 400);
				p.frameRate(60);
			};

			p.draw = () => {
				p.background(15, 23, 42); // Slate 900 background

				// 1. Draw Target (Global Minimum)
				// Creating a "contour map" effect
				p.noFill();
				for (let i = 0; i < 10; i++) {
					p.stroke(56, 189, 248, 50 - i * 5); // Blue fading out
					p.ellipse(200, 200, i * 40, i * 40);
				}

				// 2. Simulation Logic
				// Target is (200, 200). Calculate vector to target:
				let dx = 200 - position.x;
				let dy = 200 - position.y;

				// NOISE FACTOR: This is the core visual lesson.
				// Smaller batch size = Higher Noise (Stochastic Gradient Descent)
				// Larger batch size = Lower Noise (Batch Gradient Descent)
				// 128 batch = 0.5 noise, 4 batch = 15 noise
				let noiseMagnitude = (130 - batchSize) * 0.15;
				
				let noiseX = p.random(-noiseMagnitude, noiseMagnitude);
				let noiseY = p.random(-noiseMagnitude, noiseMagnitude);

				// Move the "ball"
				// We stop if we are very close to center
				if (p.dist(position.x, position.y, 200, 200) > 5) {
					position.x += (dx * learningRate) + noiseX;
					position.y += (dy * learningRate) + noiseY;
					
					// Store Path for visualization
					path.push({x: position.x, y: position.y});
					if(path.length > 200) path.shift();
				}

				// 3. Draw Path history
				p.noFill();
				// Green for stable (large batch), Pink for noisy (small batch)
				p.stroke(batchSize > 64 ? '#4ade80' : '#f472b6'); 
				p.strokeWeight(2);
				p.beginShape();
				path.forEach(pt => p.vertex(pt.x, pt.y));
				p.endShape();

				// 4. Draw Current Position (Head)
				p.fill(255);
				p.noStroke();
				p.circle(position.x, position.y, 8);
				
				// 5. UI Overlays
				p.fill(255);
				p.noStroke();
				p.textSize(12);
				p.text(`Batch Size: ${batchSize}`, 10, 20);
				p.text(`Gradient Noise: ${noiseMagnitude.toFixed(2)}`, 10, 35);
			};
		};

		myP5 = new p5(sketch, container);

		return () => {
			myP5.remove();
		};
	});
</script>

<div class="overflow-hidden rounded-xl shadow-2xl border border-slate-700 w-full flex justify-center bg-slate-900" bind:this={container}></div>