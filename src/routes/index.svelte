<script>
	import { Curtains, TextureLoader, Plane, Vec3 } from 'curtainsjs/src/index.mjs';
	import { onMount } from 'svelte';
	import anime from 'animejs';
	import vertex from '../lib/photoday.vert?raw';
	import fragment from '../lib/photoday.frag?raw';
	import { fly, fade } from 'svelte/transition';
	let canvas,
		isTrackpad = true,
		scrollPos = 0,
		curtains,
		scrollEffect = 0,
		noiseFreq = 0.9,
		noiseAmp = 0.04,
		noiseEffect = 0,
		wholeWidth,
		planes = [],
		animate,
		trans = { count: 0 },
		count = 1,
		social = [
			'instagram.com/svobodinaphoto',
			'vk.com/svobodinaphoto',
			't.me/svobodinaphoto',
			'Позвонить'
		],
		link = [
			'https://www.instagram.com/svobodinaphoto/',
			'https://vk.com/svobodinaphoto',
			'https://t.me/svobodinaphoto',
			'tel:+79514616243'
		],
		pricedata = [
			{
				title: 'Первый пакет»',
				body: [
					'Съемка 40 минут',
					' 15 фотографий с цветокоррекцией и ретушью, где она необходима',
					'Срок отдачи: 2 недели'
				],
				price: '2.000р.'
			},
			{
				title: 'Второй пакет»',
				body: [
					'Съемка 40 минут',
					' 25 фотографий с цветокоррекцией и ретушью, где она необходима',
					'Макияж от профессионального визажиста',
					'Срок отдачи: 2 недели'
				],
				price: '3.700р.'
			},
			{
				title: 'Третий пакет»',
				body: [
					'Съемка 40 минут',
					' 40-50 фотографий с цветокоррекцией и ретушью, где она необходима',
					'Макияж от профессионального визажиста',
					'Срок отдачи: 1 неделя'
				],
				price: '4.500р.'
			}
		],
		photos = [
			'2560_21-01-04-12-50-30',
			'3_x1400',
			// '2560_21-01-04-12-23-33',
			'20-09-06-13-12-33',
			'20-09-06-13-24-23',
			'20-09-06-13-22-33',
			'20-09-06-13-25-23',
			'20-08-31-12-24-31',
			'20-08-31-12-33-54_x1200',
			'20-08-31-12-01-02'
			// '20-08-31-12-16-04'
			// '3'
		];
	// ==========================

	function initPlane(planesElement) {
		for (const iterator of planesElement) {
			const plane = new Plane(curtains, iterator, {
				vertexShader: vertex,
				fragmentShader: fragment,
				widthSegments: 16,
				heightSegments: 16,
				//   autoloadSources: false,
				fov: 75,
				alwaysDraw: false,
				shareProgram: true,
				texturesOptions: {
					minFilter: curtains.gl.LINEAR_MIPMAP_NEAREST
				},
				uniforms: {
					noiseFreq: {
						name: 'uNoiseFreq',
						type: '1f',
						value: noiseFreq
					},
					noiseAmp: {
						name: 'uNoiseAmp',
						type: '1f',
						value: noiseAmp
					},
					noiseEffect: {
						name: 'uNoiseEffect',
						type: '1f',
						value: 0
					},
					scrollEffect: {
						name: 'uScrollEffect',
						type: '1f',
						value: 0
					},
					time: {
						name: 'uTime',
						type: '1f',
						value: 0
					}
				}
			});
			planes.push(plane);

			handlePlanes(plane);
		}
	}

	function initAnimate() {
		animate = anime.timeline({
			autoplay: false
		});
		animate
			.add(
				{
					targets: '.webgl',
					scale: [1.13, 1],
					opacity: [0, 1],
					easing: 'linear',
					duration: 800
				},
				0
			)
			.add(
				{
					update: () => {
						scrollEffect += (80 - scrollEffect) * 0.9;
					},
					duration: 500
				},
				0
			);
	}

	function handlePlanes(plane) {
		plane
			.onReady(() => {
				console.log(plane.relativeTranslation.x);
				console.log(plane.getWebGLBoundingRect().width, 'webjl');
				console.log(plane.getBoundingRect().width, 'bound');
				if (plane.index === planes.length - 1) {
					document.body.classList.add('planes-loaded');
					animate.play();
				}
			})
			.onRender(() => {
				noiseEffect += scrollEffect / 45000;
				plane.uniforms.time.value += 0.01;
				plane.uniforms.scrollEffect.value = scrollEffect / 100;
				plane.uniforms.noiseEffect.value = noiseEffect;

				// scale plane and its texture
				// plane.setScale(new Vec2(1, 1 + Math.abs(this.scrollEffect) / 1500))
				// plane.textures[0].setScale(
				//   new Vec2(1, 1 + Math.abs(this.scrollEffect) / 1550)
				// )
			});
	}
	function calcPos(scr, pos) {
		let temp =
			(scr + planes[0].getWebGLBoundingRect().width * photos.length) %
			(planes[0].getWebGLBoundingRect().width * photos.length);

		return temp;
	}

	function initCurtains() {
		curtains = new Curtains({
			container: canvas,
			production: true,
			pixelRatio: Math.min(1.5, window.devicePixelRatio)
		});

		curtains
			.onRender(() => {
				scrollEffect = curtains.lerp(scrollEffect, 0, 0.035);
				planes.forEach((pl) => {
					// console.log(pl.relativeTranslation.x, pl.index);
					pl.setRelativeTranslation(
						new Vec3(calcPos(scrollPos, pl.relativeTranslation.x) * 0.5, 0, 0)
					);
					// pl.setRelativeTranslation.x += scrollPos;
				});
			})
			.onScroll(() => {
				const delta = curtains.getScrollDeltas();
				delta.y = -delta.y;
				// threshold
				if (delta.y > 95) {
					delta.y = 95;
				} else if (delta.y < -95) {
					delta.y = -95;
				}

				if (Math.abs(delta.y) > Math.abs(scrollEffect)) {
					scrollEffect = curtains.lerp(scrollEffect, delta.y, 0.5);
				}
			})
			.onError(() => {
				document.body.classList.add('no-curtains', 'planes-loaded');
			})
			.onContextLost(() => {
				curtains.restoreContext();
			});
	}

	function onWheel(e) {
		// if ($eventAnimation) {
		e.preventDefault();
		if (isTrackpad) {
			if (e.wheelDeltaY) {
				if (Math.abs(e.wheelDeltaY) !== 120) {
					isTrackpad = false;
				}
			} else if (e.deltaMode === 0) {
				isTrackpad = false;
			}
		}
		const delta = window.navigator.userAgent.includes('Firefox') ? e.deltaY * 33 : e.deltaY;
		!isTrackpad ? (scrollPos += e.deltaY * -1) : (scrollPos += delta * -1);
		//   sliderState.endPosition = sliderState.currentPosition;
		//   onChangeTitle(sliderState.currentPosition, e);
		// }
		// console.log(scrollPos);
	}

	onMount(() => {
		let i = 4;
		setInterval(() => {
			i++;
			count = i % 4;
		}, 2500);
		const planeElements = document.getElementsByClassName('plane');
		initCurtains();
		initPlane(planeElements);
		initAnimate();
		// const frontPlane = new Vec3(0, 0, 80);
		// planes[1].setRelativeTranslation(frontPlane);
		// planes[3].setRelativeTranslation(frontPlane);
		// planes[8].setRelativeTranslation(frontPlane);
		// const rearPlane = new Vec3(0, 0, -80);
		// planes[0].setRelativeTranslation(rearPlane);
		// planes[2].setRelativeTranslation(rearPlane);
		// planes[7].setRelativeTranslation(frontPlane);
	});
</script>

<svelte:head>
	<meta name="author" content="Anastasia Svobodina" />
	<meta name="description" content="Photoproject" />
	<meta property="og:title" content="SvobodinaPhoto" />
	<meta property="og:type" content="website" />
	<meta property="og:image" content="/photoday/cover.jpg" />
	<meta property="og:image:type" content="image/jpeg" />
	<meta property="og:image:width" content="600" />
	<meta property="og:image:height" content="307" />
	<meta property="og:url" content="https://photoproject.svobodinaphoto.com/" />
	<meta property="og:locale" content="ru_RU" />
	<meta property="og:description" content="The best photographer ever" />
	<meta name="twitter:card" content="summary_large_image" />
	<meta property="twitter:domain" content="photoproject.svobodinaphoto.com" />
	<meta property="twitter:url" content="https://photoproject.svobodinaphoto.com/" />
	<meta name="twitter:title" content="SvobodinaPhoto" />
	<meta name="twitter:description" content="Join my photoproject" />
	<meta name="twitter:image" content="https://photoproject.svobodinaphoto.com/photoday/cover.jpg" />
	<link rel="canonical" href="https://photoproject.svobodinaphoto.com/" />
	<!-- <link rel="mask-icon" href="./favicon.svg" color=”# 000000"> -->
	<title>Photoproject</title>
	<!-- <link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
	<link
		href="https://fonts.googleapis.com/css2?family=Cormorant+Infant:wght@300&display=swap"
		rel="stylesheet"
	/> -->
</svelte:head>
<main>
	<div bind:this={canvas} class="webgl" />
	<div on:wheel={onWheel} class="plane__wrapper">
		{#each photos as photo, index (index)}
			<div class="plane plane{index}">
				<picture>
					<source srcset="./photoday/{photo}.webp" type="image/webp" />
					<img
						data-sampler="planeTexture"
						alt="SvobodinaPhoto"
						crossorigin="anonimous"
						decoding="async"
						draggable="false"
						src="./photoday/{photo}.webp"
					/>
				</picture>
			</div>
		{/each}
	</div>
</main>

<style>
	@font-face {
		font-family: 'Cormorant Infant';
		font-style: normal;
		font-weight: 300;
		src: local(''), url('/fonts/cormorant-infant-v11-cyrillic-300.woff2') format('woff2'),
			url('/fonts/cormorant-infant-v11-cyrillic-300.woff') format('woff');
		font-display: swap;
	}
	:global(body) {
		margin: 0;
	}
	:global(html) {
		font-family: 'Cormorant Infant', serif;
	}
	:root {
		--vh: 1vh;
		--row__height: calc(var(--column__width) * 1.5);
		--column__width: min(8.5vw, 140px);
	}

	.webgl {
		position: fixed;
		pointer-events: none;
		top: 0;
		right: 0;
		left: 0;
		bottom: 0;
		/* opacity: 0.3; */
	}

	.plane__wrapper {
		overflow: hidden;
		height: auto;
		display: grid;
		height: 100vh;
		justify-content: center;
		grid-auto-flow: column;
		grid-auto-columns: calc(50vh * 0.66);
		align-content: center;
		/* grid-template-columns: repeat(10, var(--column__width)); */
		/* grid-template-rows: 20px repeat(24, var(--row__height)); */
		/* background-color: #aeb4b8; */
		/* grid-gap: 2vw; */
	}
	.plane {
		height: 50vh;
		width: calc(50vh * 0.66);
	}
	.plane > picture > img {
		display: none;
		height: 100%;
		width: 100%;
		object-fit: cover;
		object-position: center;
	}
</style>
