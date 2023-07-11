<script>
	import P5 from 'p5-svelte';
	import Ml5 from '$lib/components/Ml5.svelte';
	import calImage from '$lib/images/cal.png';
	import iconsImage from '$lib/images/icons.png';
	let classifier;
	let video;
	let results = [];

	let width = 700;
	let height = 450;

	const description = {
		'Healthy Tomato': {
			title: 'Die Tomaten kannst du noch essen!',
			text: 'Die Tomate enthält neben zahlreichen Vitaminen wie A, B1, C, E und Niacin auch wichtige Mineralstoffe wie Kalium, Magnesium, Calcium und Spurenelemente. Falls du Interesse an Rezeptvorschlägen hast, die auf Tomaten und anderen Lebensmitteln basieren, die bereits in deinem Kühlschrank vorhanden sind, klicke bitte auf "weiter"!'
		},
		'Rotten Tomato': {
			title: 'Die Tomate solltest du lieber nicht essen!',
			text: 'Diese Tomate ist leider verschimmelt und sollte am besten entsorgt werden. Leider kann ich dir in dem Fall keine hilfreiche Tipps geben.'
		},
		'Healthy Banana': {
			title: 'Diese Banane kannst du noch essen.',
			text: 'Bananen enthalten verschiedene Vitamine wie Vitamin C, das zur Stärkung des Immunsystems beiträgt, sowie Vitamin B6, das wichtig für den Stoffwechsel und die Bildung roter Blutkörperchen ist. Am besten isst du sie frisch.'
		},
		'Rotten Banana': {
			title: 'Diese Banane ist überreif!',
			text: 'Überreife Bananen können für köstliche Bananenbrot oder Muffins verwendet werden. Alternativ können sie auch eingefroren und später für Smoothies oder als Basis für gesunde Nicecream verwendet werden. Falls du Interesse an Rezeptvorschlägen hast, die du aus überreifen Bananen machen kannst, klicke bitte auf weiter!'
		}
	};

	let sketch = (p5) => {
		p5.setup = () => {
			p5.createCanvas(width, height);
			video = p5.createCapture(p5.VIDEO);
			video.size(width, height);

		
			video.hide();
		};

		p5.draw = () => {
			p5.image(video, 0, 0, width, height);
		};
	};

	/**
	 * Load the image classification model or a custom one:
	 * https://learn.ml5js.org/#/reference/image-classifier?id=parameters
	 * @param domElement
	 * @param ml5
	 * @param modelReady
	 */
	const mlSketch = (domElement, ml5, modelReady) => {
		// Here you could load another image classification model
		// OR you could load your custom model from teachable machine
		classifier = ml5.imageClassifier(
			'https://teachablemachine.withgoogle.com/models/iTiQfkq-4/model.json',
			video,
			modelReady
		);
	};

	const mlReady = () => {
		classifyVideo();
	};
	const classifyVideo = () => {
		classifier.classify(gotResult);
	};

	const gotResult = (err, res) => {
		if (err) {
			console.error(err);
		} else if (res) {
			results = res;
			// console.log(results);
			classifyVideo();
		}
	};
</script>

{#if sketch}
	<div class="wrapper">
		<div class="wrapper-between">
			<img src={calImage} />
			<img src={iconsImage} height="140px" width="620px" class="image-2" />
		</div>
		<div class="wrapper-between">
			<P5 {sketch} />
			<Ml5 {mlSketch} domElement={video} {mlReady} />

			<div class="result-card">
				{#each results as result}
					{#if result.confidence > 0.7}
						<div class="text-container">
							<h1>{description[result.label].title}</h1>
							<p>{description[result.label].text}</p>
							<div class="button-wrapper">
								<button>Weiter</button>
							</div>
						</div>
					{/if}
				{/each}
			</div>
		</div>
	</div>
{/if}

<style>
	.wrapper {
		display: flex;
		flex-direction: column;
		gap: 40px;
	}
	.wrapper-between {
		display: flex;
		justify-content: space-between;
	}
	.result-card {
		display: flex;
		margin-left: 160px;
		background-color: #232323;
		border-radius: 20px;
		justify-content: flex-end;
	}
	.image-2 {
		margin-left: 160px;
		margin-top: 65px;
	}
	.text-container {
		width: 500px;
		display: flex;
		flex-direction: column;
	}

	.button-wrapper {
		position: absolute;
		bottom: 150px;
		margin-left: 350px;
	}

	h1 {
		padding: 30px;
		text-align: left;
		margin-bottom: -50px;
		word-break: break-word;
	}
</style>
