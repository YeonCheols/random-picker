<script>
	import Roulette from '$lib/components/Roulette.svelte';

	let items = ['피자', '치킨', '햄버거', '스시', '파스타', '샐러드'];
	let newItem = '';
	let spinning = false;
	let result = '';
	let showResult = false;
	let rouletteComponent;

	function addItem() {
		if (newItem.trim() && !items.includes(newItem.trim())) {
			items = [...items, newItem.trim()];
			newItem = '';
		}
	}

	function removeItem(index) {
		items = items.filter((_, i) => i !== index);
	}

	function spinRoulette() {
		if (items.length === 0) return;

		spinning = true;
		showResult = false;
		result = '';

		// Call the spin function on the roulette component
		if (rouletteComponent) {
			rouletteComponent.spin();
		}
	}

	function handleSpinComplete(selectedItem) {
		result = selectedItem;
		showResult = true;
		spinning = false;
	}

	function clearAll() {
		items = [];
		result = '';
		showResult = false;
	}

	function addRandomItems() {
		const sampleItems = [
			'피자',
			'치킨',
			'햄버거',
			'스시',
			'파스타',
			'샐러드',
			'떡볶이',
			'김치찌개',
			'갈비찜',
			'순두부찌개',
			'비빔밥',
			'라멘',
			'우동',
			'덮밥',
			'카레',
			'스테이크',
			'샌드위치',
			'토스트',
			'아이스크림',
			'케이크',
			'커피',
			'차',
			'주스'
		];

		const shuffled = sampleItems.sort(() => 0.5 - Math.random());
		items = shuffled.slice(0, 6);
	}

	// Handle Enter key for adding items
	function handleKeyPress(event) {
		if (event.key === 'Enter') {
			addItem();
		}
	}
</script>

<svelte:head>
	<title>룰렛 랜덤 픽커</title>
	<meta name="description" content="재미있는 룰렛으로 랜덤 선택을 해보세요!" />
</svelte:head>

<div class="roulette-container">
	<h1 class="title">🎯 룰렛 랜덤 픽커</h1>

	<!-- Roulette Component -->
	<Roulette bind:this={rouletteComponent} {items} {spinning} onSpinComplete={handleSpinComplete} />

	<!-- Controls -->
	<div class="controls">
		<!-- Add Item -->
		<div class="input-group">
			<label for="newItem">새 항목 추가</label>
			<input
				id="newItem"
				type="text"
				bind:value={newItem}
				on:keypress={handleKeyPress}
				placeholder="항목을 입력하세요"
				disabled={spinning}
			/>
		</div>

		<button class="btn btn-secondary" on:click={addItem} disabled={spinning || !newItem.trim()}>
			추가
		</button>

		<button
			class="btn btn-primary"
			on:click={spinRoulette}
			disabled={spinning || items.length === 0}
		>
			{spinning ? '돌리는 중...' : '룰렛 돌리기'}
		</button>

		<button class="btn btn-secondary" on:click={addRandomItems} disabled={spinning}>
			예시 항목 추가
		</button>

		<button class="btn btn-secondary" on:click={clearAll} disabled={spinning}> 모두 지우기 </button>
	</div>

	<!-- Items List -->
	{#if items.length > 0}
		<div class="items-section">
			<h3>현재 항목들 ({items.length}개)</h3>
			<div class="items-list">
				{#each items as item, index}
					<div class="item-tag">
						<span>{item}</span>
						<button class="remove-btn" on:click={() => removeItem(index)} disabled={spinning}>
							×
						</button>
					</div>
				{/each}
			</div>
		</div>
	{/if}

	<!-- Result Display -->
	{#if showResult && result}
		<div class="result fade-in bounce">
			🎉 결과: <strong>{result}</strong> 🎉
		</div>
	{/if}

	<!-- Instructions -->
	<div class="instructions">
		<h3>사용법</h3>
		<ul>
			<li>위의 입력창에 항목을 입력하고 "추가" 버튼을 클릭하세요</li>
			<li>"예시 항목 추가" 버튼으로 빠르게 항목을 추가할 수 있습니다</li>
			<li>"룰렛 돌리기" 버튼을 클릭하면 룰렛이 회전합니다</li>
			<li>룰렛이 멈추면 선택된 항목이 표시됩니다</li>
		</ul>
	</div>
</div>

<style>
	.items-section {
		margin-top: 2rem;
		text-align: center;
	}

	.items-section h3 {
		margin-bottom: 1rem;
		font-size: 1.5rem;
	}

	.items-list {
		display: flex;
		flex-wrap: wrap;
		gap: 0.5rem;
		justify-content: center;
		max-width: 600px;
		margin: 0 auto;
	}

	.item-tag {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		background: rgba(255, 255, 255, 0.2);
		padding: 0.5rem 1rem;
		border-radius: 2rem;
		border: 1px solid rgba(255, 255, 255, 0.3);
		backdrop-filter: blur(10px);
		transition: all 0.3s ease;
	}

	.item-tag:hover {
		background: rgba(255, 255, 255, 0.3);
		transform: translateY(-2px);
	}

	.remove-btn {
		background: rgba(255, 107, 107, 0.8);
		color: white;
		border: none;
		border-radius: 50%;
		width: 20px;
		height: 20px;
		display: flex;
		align-items: center;
		justify-content: center;
		cursor: pointer;
		font-size: 14px;
		font-weight: bold;
		transition: all 0.3s ease;
	}

	.remove-btn:hover {
		background: rgba(255, 107, 107, 1);
		transform: scale(1.1);
	}

	.remove-btn:disabled {
		opacity: 0.5;
		cursor: not-allowed;
		transform: none;
	}

	.instructions {
		margin-top: 3rem;
		padding: 2rem;
		background: rgba(255, 255, 255, 0.1);
		border-radius: 1rem;
		backdrop-filter: blur(10px);
		border: 1px solid rgba(255, 255, 255, 0.2);
		max-width: 600px;
	}

	.instructions h3 {
		margin-bottom: 1rem;
		font-size: 1.3rem;
		text-align: center;
	}

	.instructions ul {
		text-align: left;
		line-height: 1.6;
	}

	.instructions li {
		margin-bottom: 0.5rem;
	}

	/* Responsive adjustments */
	@media (max-width: 768px) {
		.items-list {
			flex-direction: column;
			align-items: center;
		}

		.item-tag {
			width: 100%;
			max-width: 300px;
			justify-content: space-between;
		}
	}
</style>
