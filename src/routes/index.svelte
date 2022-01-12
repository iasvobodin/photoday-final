<script>
	import { Curtains, TextureLoader, Plane, Vec3 } from 'curtainsjs/src/index.mjs';
	import { onMount } from 'svelte';
	import anime from 'animejs';
	import vertex from '../lib/photoday.vert?raw';
	import fragment from '../lib/photoday.frag?raw';
	import { fly, fade } from 'svelte/transition';
	let canvas,
		curtains,
		scrollEffect = 0,
		noiseFreq = 0.9,
		noiseAmp = 0.04,
		noiseEffect = 0,
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

	function initCurtains() {
		curtains = new Curtains({
			container: canvas,
			production: true,
			pixelRatio: Math.min(1.5, window.devicePixelRatio)
		});

		curtains
			.onRender(() => {
				scrollEffect = curtains.lerp(scrollEffect, 0, 0.035);
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
		const frontPlane = new Vec3(0, 0, 80);
		planes[1].setRelativeTranslation(frontPlane);
		planes[3].setRelativeTranslation(frontPlane);
		planes[8].setRelativeTranslation(frontPlane);
		const rearPlane = new Vec3(0, 0, -80);
		planes[0].setRelativeTranslation(rearPlane);
		planes[2].setRelativeTranslation(rearPlane);
		planes[7].setRelativeTranslation(frontPlane);
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
	<section class=" content first">
		<div class="photo">
			<picture>
				<source srcset="./photoday/fw.jpg" type="image/webp" />
				<img
					data-sampler="planeTexture"
					alt="SvobodinaPhoto"
					crossorigin="anonimous"
					decoding="async"
					draggable="false"
					src="./photoday/fw.jpg"
				/>
			</picture>
		</div>
		<div class="header">
			<h1>Title</h1>
			<p>Lorem ipsum dolor sit amet.</p>
		</div>
	</section>
	<section class=" content second">
		<div class="photo">
			<picture>
				<source srcset="./photoday/bg.jpg" type="image/webp" />
				<img
					data-sampler="planeTexture"
					alt="SvobodinaPhoto"
					crossorigin="anonimous"
					decoding="async"
					draggable="false"
					src="./photoday/bg.jpg"
				/>
			</picture>
		</div>
		<div class="decsription">
			<h2>Title</h2>
			<p>
				<b>
					Lorem, ipsum dolor sit amet consectetur adipisicing elit. Aliquid dignissimos totam, quas,
					dolor molestias adipisci eveniet ex ipsa quod sequi tempore sapiente facere eligendi
					<br /><br />
					placeat cupiditate voluptatum, explicabo accusamus quasi quia delectus eaque voluptas quibusdam?
				</b>
			</p>
		</div>
	</section>
	<section class=" content third">
		<div class="price">
			<h2>Lorem ipsum dolor sit amet consectetur.</h2>
		</div>
		<div class="table">
			<div class="ff">
				<p>
					Lorem ipsum, dolor sit amet consectetur adipisicing elit. Error, ab perferendis
					repellendus magnam nemo aliquid.
				</p>
			</div>
			<div class="ss">
				<p>
					Lorem ipsum, dolor sit amet consectetur adipisicing elit. Blanditiis dolores labore
					eveniet, ad accusamus corporis.
				</p>
			</div>
		</div>
	</section>
	<section class=" content fourth">
		<div class="photo">
			<picture>
				<source srcset="./photoday/fw.jpg" type="image/webp" />
				<img
					data-sampler="planeTexture"
					alt="SvobodinaPhoto"
					crossorigin="anonimous"
					decoding="async"
					draggable="false"
					src="./photoday/fw.jpg"
				/>
			</picture>
		</div>
		<div class="sliderdescription">
			<h1>Slider</h1>
			<!-- <p>Lorem ipsum dolor sit amet.</p> -->
		</div>
	</section>
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
		font-family: 'Cormorant Infant', serif;
	}
	.content {
		min-height: 100vh;
		border: 1px solid pink;
		display: grid;
	}
	.first {
		grid-auto-flow: row;
	}
	.first .photo {
		place-self: center;
		height: calc(90vw * 1.5);
		width: 90vw;
		border-radius: 5px;
		overflow: hidden;
	}
	picture {
		width: inherit;
		height: inherit;
	}
	img {
		width: 100%;
		height: 100%;
		object-position: center;
		object-fit: cover;
	}
	.first .header {
		align-self: center;
		margin-left: 3ch;
	}
	p {
		font-size: max(26px, 2.7vw);
	}
	h1 {
		font-size: max(40px, 5vw);
		margin: 0;
	}
	h2 {
		font-size: max(32px, 4vw);
		margin: 0;
	}
	.decsription {
		border-radius: 5px;
		overflow: hidden;
		width: 85%;
		background-color: rgba(255, 255, 255, 0.7);
		position: absolute;
		place-self: center;
		padding: 7vw 3vw;
	}
	.table {
		display: grid;
		grid-auto-flow: column;
	}
	.ff,
	.ss {
		padding: 10px 3vh;
	}
	.price {
		place-self: center;
		max-height: 20vh;
	}
	.price h2 {
		text-align: center;
	}
	.sliderdescription {
		text-align: center;
	}
</style>
