<script>
	import { onMount, tick } from 'svelte';

	export let items = [];
	export let spinning = false;
	export let onSpinComplete = () => {};

	// Expose spin function
	export function spin() {
		if (isSpinning) return;

		isSpinning = true;
		spinning = true;

		// Random spin duration and speed
		const spinDuration = 3000 + Math.random() * 2000; // 3-5 seconds
		const totalRotation = 5 + Math.random() * 5; // 5-10 full rotations

		// Calculate target rotation
		targetRotation = currentRotation + totalRotation * 2 * Math.PI;

		// Set initial speed
		spinSpeed = maxSpinSpeed;

		// Gradually increase friction to slow down naturally
		const frictionInterval = setInterval(() => {
			friction *= 0.999;
			if (friction < 0.95) {
				clearInterval(frictionInterval);
			}
		}, 50);
	}

	let canvas;
	let ctx;
	let animationId;
	let currentRotation = 0;
	let targetRotation = 0;
	let spinSpeed = 0;
	let maxSpinSpeed = 0.3;
	let friction = 0.98;
	let isSpinning = false;

	// Canvas dimensions
	const CANVAS_SIZE = 400;
	const CENTER = CANVAS_SIZE / 2;
	const RADIUS = 150;
	const WHEEL_THICKNESS = 20;

	// Colors for wheel segments - more vibrant and contrasting
	const colors = [
		'#FF6B6B', // 빨강
		'#4ECDC4', // 청록
		'#45B7D1', // 파랑
		'#96CEB4', // 연두
		'#FFEAA7', // 노랑
		'#DDA0DD', // 연보라
		'#FF8A80', // 주황
		'#81C784', // 초록
		'#64B5F6', // 하늘색
		'#F06292', // 분홍
		'#FFD54F', // 금색
		'#A1887F' // 갈색
	];

	// Pointer properties
	const pointerLength = 30;
	const pointerWidth = 8;

	onMount(() => {
		if (canvas) {
			ctx = canvas.getContext('2d');
			if (ctx) {
				drawWheel();
				startAnimation();
			}
		}
	});

	function startAnimation() {
		function animate() {
			updateWheel();
			drawWheel();
			animationId = requestAnimationFrame(animate);
		}
		animate();
	}

	function updateWheel() {
		if (isSpinning) {
			currentRotation += spinSpeed;
			spinSpeed *= friction;

			// Stop spinning when speed is very low
			if (Math.abs(spinSpeed) < 0.001) {
				isSpinning = false;
				spinning = false;

				// Calculate which item was selected
				const normalizedRotation =
					((currentRotation % (2 * Math.PI)) + 2 * Math.PI) % (2 * Math.PI);
				const segmentAngle = (2 * Math.PI) / items.length;
				const selectedIndex = Math.floor(
					((2 * Math.PI - normalizedRotation) / segmentAngle) % items.length
				);
				const selectedItem = items[selectedIndex];

				// Trigger callback
				onSpinComplete(selectedItem);
			}
		}
	}

	function drawWheel() {
		if (!ctx) return;

		// Clear canvas
		ctx.clearRect(0, 0, CANVAS_SIZE, CANVAS_SIZE);

		// Draw wheel background
		ctx.save();
		ctx.translate(CENTER, CENTER);
		ctx.rotate(currentRotation);

		// Draw wheel segments
		const segmentAngle = (2 * Math.PI) / items.length;

		// First pass: draw all segments
		items.forEach((item, index) => {
			const startAngle = index * segmentAngle;
			const endAngle = startAngle + segmentAngle;

			// Draw segment
			ctx.beginPath();
			ctx.moveTo(0, 0);
			ctx.arc(0, 0, RADIUS, startAngle, endAngle);
			ctx.closePath();

			// Fill segment
			ctx.fillStyle = colors[index % colors.length];
			ctx.fill();

			// Add gradient effect for better depth
			const gradient = ctx.createRadialGradient(0, 0, 0, 0, 0, RADIUS);
			gradient.addColorStop(0, 'rgba(255, 255, 255, 0.4)');
			gradient.addColorStop(0.7, 'rgba(255, 255, 255, 0.1)');
			gradient.addColorStop(1, 'rgba(0, 0, 0, 0.3)');
			ctx.fillStyle = gradient;
			ctx.fill();

			// Draw segment border
			ctx.strokeStyle = 'rgba(255, 255, 255, 0.9)';
			ctx.lineWidth = 3;
			ctx.stroke();

			// Add inner border for better separation
			ctx.strokeStyle = 'rgba(0, 0, 0, 0.3)';
			ctx.lineWidth = 1;
			ctx.stroke();
		});

		// Second pass: draw all text labels
		items.forEach((item, index) => {
			const startAngle = index * segmentAngle;

			// Draw text background for better readability
			ctx.save();
			ctx.rotate(startAngle + segmentAngle / 2);

			// Set font first
			ctx.font = 'bold 20px "Malgun Gothic", "맑은 고딕", sans-serif';

			// Truncate text if too long
			const maxWidth = RADIUS * 0.6;
			let text = item;
			while (ctx.measureText(text).width > maxWidth && text.length > 1) {
				text = text.slice(0, -1) + '...';
			}

			// Measure text dimensions
			const textMetrics = ctx.measureText(text);
			const textWidth = textMetrics.width;
			const textHeight = 24; // Approximate text height

			// Draw background rectangle
			const padding = 8;
			const bgWidth = textWidth + padding * 2;
			const bgHeight = textHeight + padding * 2;
			const bgX = -bgWidth / 2;
			const bgY = -RADIUS * 0.6 - bgHeight / 2;

			// Background with rounded corners effect
			ctx.fillStyle = 'rgba(255, 255, 255, 0.9)';
			ctx.shadowColor = 'rgba(0, 0, 0, 0.3)';
			ctx.shadowBlur = 4;
			ctx.shadowOffsetX = 2;
			ctx.shadowOffsetY = 2;
			ctx.fillRect(bgX, bgY, bgWidth, bgHeight);

			// Draw text
			ctx.textAlign = 'center';
			ctx.textBaseline = 'middle';
			ctx.fillStyle = 'rgba(0, 0, 0, 0.8)';
			ctx.shadowColor = 'rgba(255, 255, 255, 0.5)';
			ctx.shadowBlur = 1;
			ctx.shadowOffsetX = 0;
			ctx.shadowOffsetY = 0;

			// Position text in the exact center of each segment
			ctx.fillText(text, 0, -RADIUS * 0.6);
			ctx.restore();
		});

		ctx.restore();

		// Draw center circle
		ctx.save();
		ctx.translate(CENTER, CENTER);

		// Center gradient
		const centerGradient = ctx.createRadialGradient(0, 0, 0, 0, 0, 30);
		centerGradient.addColorStop(0, 'rgba(255, 255, 255, 0.9)');
		centerGradient.addColorStop(1, 'rgba(255, 255, 255, 0.3)');

		ctx.fillStyle = centerGradient;
		ctx.beginPath();
		ctx.arc(0, 0, 30, 0, 2 * Math.PI);
		ctx.fill();

		// Center border
		ctx.strokeStyle = 'rgba(255, 255, 255, 0.8)';
		ctx.lineWidth = 3;
		ctx.stroke();

		ctx.restore();

		// Draw pointer
		drawPointer();
	}

	function drawPointer() {
		if (!ctx) return;

		ctx.save();
		ctx.translate(CENTER, 25);

		// Pointer shadow
		ctx.shadowColor = 'rgba(0, 0, 0, 0.7)';
		ctx.shadowBlur = 8;
		ctx.shadowOffsetX = 3;
		ctx.shadowOffsetY = 3;

		// Pointer gradient
		const pointerGradient = ctx.createLinearGradient(0, 0, 0, pointerLength);
		pointerGradient.addColorStop(0, '#FF6B6B');
		pointerGradient.addColorStop(1, '#EE5A24');

		ctx.fillStyle = pointerGradient;
		ctx.beginPath();
		ctx.moveTo(-pointerWidth / 2, 0);
		ctx.lineTo(pointerWidth / 2, 0);
		ctx.lineTo(0, pointerLength);
		ctx.closePath();
		ctx.fill();

		// Pointer border
		ctx.strokeStyle = 'rgba(255, 255, 255, 0.8)';
		ctx.lineWidth = 2;
		ctx.stroke();

		ctx.restore();
	}

	// Watch for items changes and redraw
	$: if (items.length > 0 && ctx) {
		drawWheel();
	}
</script>

<div class="roulette-wrapper">
	<canvas bind:this={canvas} width={CANVAS_SIZE} height={CANVAS_SIZE} class="roulette-canvas" />

	{#if items.length === 0}
		<div class="no-items">
			<p>룰렛에 추가할 항목이 없습니다.</p>
		</div>
	{/if}
</div>

<style>
	.roulette-wrapper {
		position: relative;
		display: flex;
		justify-content: center;
		align-items: center;
		margin: 2rem 0;
	}

	.roulette-canvas {
		border-radius: 50%;
		box-shadow: 0 0 30px rgba(0, 0, 0, 0.3), inset 0 0 30px rgba(255, 255, 255, 0.1);
		background: radial-gradient(circle, rgba(255, 255, 255, 0.1) 0%, transparent 70%);
		transition: transform 0.3s ease;
	}

	.roulette-canvas:hover {
		transform: scale(1.02);
	}

	.no-items {
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		text-align: center;
		color: rgba(255, 255, 255, 0.7);
		font-size: 1.2rem;
	}

	/* Spinning animation for the entire wheel */
	.roulette-wrapper:has(.spinning) {
		animation: wheelGlow 0.5s ease-in-out infinite alternate;
	}

	@keyframes wheelGlow {
		from {
			filter: brightness(1);
		}
		to {
			filter: brightness(1.2);
		}
	}
</style>
