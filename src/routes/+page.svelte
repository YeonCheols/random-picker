<script>
	import Roulette from '$lib/components/Roulette.svelte';

	let items = ['피자', '치킨', '햄버거', '스시', '파스타', '샐러드'];
	let newItem = '';
	let spinning = false;
	let result = '';
	let showResult = false;
	let rouletteComponent;
	let resultImage = '';
	let generatingImage = false;

	// Gemini API 키 (실제 사용시 환경변수로 관리)
	const GEMINI_API_KEY = import.meta.env.VITE_GEMINI_API_KEY || 'YOUR_GEMINI_API_KEY';

	async function generateFoodImage(foodName) {
		console.log('이미지 생성 시작:', foodName);
		console.log('API 키 확인:', GEMINI_API_KEY ? '설정됨' : '설정되지 않음');

		if (!GEMINI_API_KEY || GEMINI_API_KEY === 'YOUR_GEMINI_API_KEY') {
			console.warn('Gemini API 키가 설정되지 않았습니다.');
			return null;
		}

		try {
			generatingImage = true;

			// 한국어 음식명을 영어로 변환 (간단한 매핑)
			const foodMapping = {
				피자: 'pizza',
				치킨: 'fried chicken',
				햄버거: 'hamburger',
				스시: 'sushi',
				파스타: 'pasta',
				샐러드: 'salad',
				떡볶이: 'tteokbokki',
				김치찌개: 'kimchi stew',
				갈비찜: 'braised ribs',
				순두부찌개: 'soft tofu stew',
				비빔밥: 'bibimbap',
				라멘: 'ramen',
				우동: 'udon',
				덮밥: 'rice bowl',
				카레: 'curry',
				스테이크: 'steak',
				샌드위치: 'sandwich',
				토스트: 'toast',
				아이스크림: 'ice cream',
				케이크: 'cake',
				커피: 'coffee',
				차: 'tea',
				주스: 'juice'
			};

			const englishFoodName = foodMapping[foodName] || foodName;
			const prompt = `delicious ${englishFoodName}, high quality food photography, appetizing, professional lighting, on a clean plate, realistic`;

			// 사용 가능한 모델 확인만 먼저 실행
			console.log('사용 가능한 모델 확인 중...');
			const modelsResponse = await fetch(
				`https://generativelanguage.googleapis.com/v1beta/models?key=${GEMINI_API_KEY}`,
				{
					method: 'GET',
					headers: {
						'Content-Type': 'application/json'
					}
				}
			);

			const modelsData = await modelsResponse.json();
			console.log('사용 가능한 모델들:', modelsData);

			// 모델 목록 확인 후 이미지 생성은 일단 주석 처리
			/*
			// Gemini 이미지 생성 API 호출 (Imagen 모델 사용)
			const response = await fetch(
				`https://generativelanguage.googleapis.com/v1beta/models/imagen-3.1:generateContent?key=${GEMINI_API_KEY}`,
				{
					method: 'POST',
					headers: {
						'Content-Type': 'application/json'
					},
					body: JSON.stringify({
						contents: [
							{
								parts: [
									{
										text: `A delicious ${englishFoodName}, high quality food photography, appetizing, professional lighting, on a clean plate, realistic, 4K resolution`
									}
								]
							}
						],
						generationConfig: {
							temperature: 0.7,
							topK: 40,
							topP: 0.95
						}
					})
				}
			);

			const data = await response.json();
			console.log('Gemini API 응답:', data);
			*/

			// 응답에서 이미지 데이터 추출
			if (data.candidates && data.candidates[0] && data.candidates[0].content) {
				const parts = data.candidates[0].content.parts;
				for (const part of parts) {
					if (part.inlineData && part.inlineData.mimeType.startsWith('image/')) {
						// base64 이미지 데이터를 data URL로 변환
						resultImage = `data:${part.inlineData.mimeType};base64,${part.inlineData.data}`;
						break;
					}
				}
			}
		} catch (error) {
			console.error('이미지 생성 중 오류:', error);
			console.error('에러 상세:', error.message);
			console.error('에러 스택:', error.stack);
			resultImage = '';
		} finally {
			generatingImage = false;
		}
	}

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
		console.log('spinRoulette 호출됨');
		if (items.length === 0) return;

		spinning = true;
		showResult = false;
		result = '';
		resultImage = '';

		// Call the spin function on the roulette component
		if (rouletteComponent) {
			console.log('rouletteComponent.spin() 호출');
			rouletteComponent.spin();
		} else {
			console.log('rouletteComponent가 없음');
		}
	}

	async function handleSpinComplete(selectedItem) {
		console.log('handleSpinComplete 호출됨:', selectedItem);
		result = selectedItem;
		showResult = true;
		spinning = false;

		// 선택된 아이템의 이미지 생성
		console.log('generateFoodImage 호출 전');
		await generateFoodImage(selectedItem);
		console.log('generateFoodImage 호출 후');
	}

	function clearAll() {
		items = [];
		result = '';
		showResult = false;
		resultImage = '';
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

		{#if generatingImage}
			<div class="result-image loading">
				<div class="loading-spinner" />
				<p>🍽️ {result} 이미지를 생성 중입니다...</p>
			</div>
		{:else if resultImage}
			<div class="result-image fade-in">
				<img src={resultImage} alt={result} />
				<p class="image-caption">✨ AI가 생성한 {result} 이미지</p>
			</div>
		{:else}
			<div class="result-image fallback">
				<div class="food-emoji">🍽️</div>
				<p>이미지 생성에 실패했습니다</p>
			</div>
		{/if}
	{/if}

	<!-- Instructions -->
	<div class="instructions">
		<h3>사용법</h3>
		<ul>
			<li>위의 입력창에 항목을 입력하고 "추가" 버튼을 클릭하세요</li>
			<li>"예시 항목 추가" 버튼으로 빠르게 항목을 추가할 수 있습니다</li>
			<li>"룰렛 돌리기" 버튼을 클릭하면 룰렛이 회전합니다</li>
			<li>룰렛이 멈추면 선택된 항목과 AI 생성 이미지가 표시됩니다</li>
		</ul>

		{#if !GEMINI_API_KEY || GEMINI_API_KEY === 'YOUR_GEMINI_API_KEY'}
			<div class="api-warning">
				<h4>⚠️ Gemini API 설정 필요</h4>
				<p>AI 이미지 생성 기능을 사용하려면:</p>
				<ol>
					<li>
						<a href="https://aistudio.google.com/app/apikey" target="_blank">Google AI Studio</a
						>에서 API 키 발급
					</li>
					<li>프로젝트 루트에 <code>.env</code> 파일 생성</li>
					<li><code>VITE_GEMINI_API_KEY=your_api_key_here</code> 추가</li>
				</ol>
				<p><strong>참고:</strong> Gemini API는 이미지 생성 기능을 지원합니다.</p>
			</div>
		{/if}
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

	/* Result Image Styles */
	.result-image {
		margin-top: 2rem;
		text-align: center;
		padding: 2rem;
		background: rgba(255, 255, 255, 0.1);
		border-radius: 1rem;
		backdrop-filter: blur(10px);
		border: 1px solid rgba(255, 255, 255, 0.2);
		max-width: 500px;
		margin-left: auto;
		margin-right: auto;
	}

	.result-image img {
		max-width: 100%;
		height: auto;
		border-radius: 0.5rem;
		box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
		transition: transform 0.3s ease;
	}

	.result-image img:hover {
		transform: scale(1.05);
	}

	.image-caption {
		margin-top: 1rem;
		font-size: 0.9rem;
		color: rgba(255, 255, 255, 0.8);
		font-style: italic;
	}

	.loading-spinner {
		width: 40px;
		height: 40px;
		border: 4px solid rgba(255, 255, 255, 0.3);
		border-top: 4px solid #ff6b6b;
		border-radius: 50%;
		animation: spin 1s linear infinite;
		margin: 0 auto 1rem;
	}

	@keyframes spin {
		0% {
			transform: rotate(0deg);
		}
		100% {
			transform: rotate(360deg);
		}
	}

	.food-emoji {
		font-size: 4rem;
		margin-bottom: 1rem;
		animation: bounce 2s infinite;
	}

	@keyframes bounce {
		0%,
		20%,
		50%,
		80%,
		100% {
			transform: translateY(0);
		}
		40% {
			transform: translateY(-10px);
		}
		60% {
			transform: translateY(-5px);
		}
	}

	.fallback p {
		color: rgba(255, 255, 255, 0.7);
		font-size: 1rem;
	}

	.api-warning {
		margin-top: 2rem;
		padding: 1.5rem;
		background: rgba(255, 193, 7, 0.1);
		border: 1px solid rgba(255, 193, 7, 0.3);
		border-radius: 0.5rem;
	}

	.api-warning h4 {
		color: #ffc107;
		margin-bottom: 1rem;
	}

	.api-warning ol {
		text-align: left;
		margin-top: 1rem;
	}

	.api-warning li {
		margin-bottom: 0.5rem;
	}

	.api-warning code {
		background: rgba(255, 255, 255, 0.1);
		padding: 0.2rem 0.4rem;
		border-radius: 0.3rem;
		font-family: monospace;
	}

	.api-warning a {
		color: #4ecdc4;
		text-decoration: none;
	}

	.api-warning a:hover {
		text-decoration: underline;
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

		.result-image {
			margin: 1rem;
			padding: 1rem;
		}

		.food-emoji {
			font-size: 3rem;
		}
	}
</style>
