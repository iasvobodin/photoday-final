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
	<meta property="og:image:width" content="300" />
	<meta property="og:image:height" content="300" />
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
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
	<link
		href="https://fonts.googleapis.com/css2?family=Cormorant+Infant:wght@300&display=swap"
		rel="stylesheet"
	/>
</svelte:head>
<main>
	<div bind:this={canvas} class="webgl" />
	<!-- <div class="top half">
		<h2>Svobodina</h2>
	</div>
	<div class="bottom half">
		<h2>Photo</h2>
	</div> -->
	<div class="plane__wrapper">
		<h1 class="main__hedlain">Фотопроект<br />"Ничего лишнего"</h1>
		<p class="description part1">
			Фотографии на белом фоне, где главный герой вы, а не дизайн фотостудии
		</p>
		<p class="description part2">Только ваши эмоции и чувства</p>
		<div class="description part4">
			<p class="date">Дата</p>
			<p class="date">03.10.2021г.*</p>
		</div>

		<div class="whom">
			<p class="description">Подходит для:</p>
			<ul>
				<li class="description2">Семейной фотосессии</li>
				<li class="description2">Контент съёмки для соцсетей</li>
				<li class="description2">Бизнес портретов</li>
			</ul>
		</div>
		<div class="where">
			<p class="description date">Место</p>
			<p class="description">Фотостудия в центре города <br /> ул. Клары Цеткин 11, 4 зал</p>
		</div>

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
	<section class="price">
		<h2 class="description date">Цены</h2>
		<div class="price__holder">
			{#each pricedata as el}
				<div class="price__tarif">
					<h3>{el.title}</h3>
					<ul style="align-self: stretch;">
						{#each el.body as element}
							<li>{element}</li>
						{/each}
					</ul>
					<h4 style="align-self: end;justify-self: center;">
						Цена : {el.price}
					</h4>
				</div>
			{/each}
		</div>
	</section>
	<div class="contact">
		<!-- <h2 class="description date">Контакты</h2> -->
		<a class="description3" href={link[count]} target="_blank"> <p>{social[count]}</p> </a>
		<br />
		<p>
			*Если вам не подошла дата или время - не расстраивайтесь и всё равно напишите мне, думаю мы
			обязательно сделаем повтор, и тогда я напишу вам в первую очередь.
		</p>
		<br />
	</div>
</main>

<style>
	:global(body) {
		margin: 0;
	}
	a {
		text-decoration: none;
		color: black;
		width: 100%;
		text-align: right;
	}
	a:hover {
		color: rgb(165, 0, 165);
	}
	:global(html) {
		font-family: 'Cormorant Infant', serif;
	}
	:root {
		--vh: 1vh;
		--row__height: calc(var(--column__width) * 1.5);
		--column__width: min(8.5vw, 140px);
	}
	h2 {
		/* color: white; */
		font-size: max(32px, 8vw);
		font-weight: 300;
		margin: 0;
		text-align: center;
	}
	h3,
	h4 {
		margin: 0;
	}
	p {
		margin: 0;
	}
	.half {
		background-color: black;
		width: 100%;
		height: calc(50 * var(--vh));
		position: fixed;
		display: grid;
		grid-template-columns: 1fr 1fr;
	}
	/* .bottom h2 {
		grid-column: 2/3;
	}
	.top h2 {
		justify-self: end;
		align-self: end;
		grid-column: 1/2;
	} */
	.bottom {
		top: calc(50 * var(--vh));
	}
	.main__hedlain {
		margin: 0;
		z-index: 2;
		grid-row: 3 / 4;
		grid-column: 6/11;
		font-size: max(40px, 5vw);
		font-family: 'Cormorant Infant', serif;
		font-weight: 300;
	}
	.description {
		font-size: max(26px, 2.7vw);
		margin: 0;
		z-index: 2;
	}
	.description2 {
		font-size: max(20px, 2vw);
		margin: 0;
	}
	.description3 {
		font-size: max(24px, 5vw);
		margin: 0;
	}
	.part1 {
		grid-row: 7 / 8;
		grid-column: 1/6;
		align-self: center;
	}
	.part2 {
		grid-row: 9 / 10;
		grid-column: 6/11;
		margin-top: calc(3 * var(--vh));
		align-self: start;
	}
	.part4 {
		grid-row: 18 / 19;
		grid-column: 1/5;
		font-size: max(32px, 32px + 1.7vw);
		align-self: start;
		transform: translateY(-30%);
	}
	.date {
		font-size: max(24px, 5vw);
	}
	.whom {
		grid-row: 11 / 13;
		grid-column: 8/11;
		transform: translateY(-30%);
	}
	.part4 > p {
		text-align: center;
	}
	.where {
		grid-row: 24 / 26;
		grid-column: 6/11;
		height: 100%;
		width: 100%;
		object-fit: cover;
		object-position: center;
	}
	.where > p {
		text-align: center;
	}
	.price {
		grid-row: 26 / 30;
		grid-column: 1/11;
		align-self: center;
	}
	.price__holder {
		display: grid;
		justify-content: center;
		grid-template-columns: repeat(auto-fit, minmax(max(22vw, 250px), 1fr));
		row-gap: calc(5 * var(--vh));
		column-gap: max(5px, 2vw);
		width: min(1400px, 95vw);
		margin: auto;
		margin-top: calc(5 * var(--vh));
	}

	.price__tarif {
		position: relative;
		display: grid;
		grid-template-rows: 10% min(calc(30 * var(--vh)) 600px) 10%;
		border-radius: 5px;
		/* padding:calc( 3 * var(--vh)) 1vw; */
	}
	.price__tarif > h3 {
		font-size: max(24px, 2.7vw);
		align-self: start;
		justify-self: center;
		white-space: pre-wrap;
		text-align: center;
	}
	.price__tarif > ul {
		align-self: stretch;
		margin-top: calc(5 * var(--vh));
		margin-bottom: calc(5 * var(--vh));
	}
	li {
		font-size: max(18px, 1.5vw);
		font-weight: 400;
		/* list-style-type: '📷'; */
		/* padding-inline-start: 1ch; */
	}
	.price__tarif > h4 {
		font-size: max(24px, 2.7vw);
		place-self: center;
		white-space: pre-wrap;
	}
	.contact {
		width: 98%;
		margin-top: calc(30 * var(--vh));
		margin-right: 2vw;
	}
	.contact > p {
		text-align: center;
		font-size: max(16px, 1.35vw);
	}
	.animate__contact {
		font-size: 56px;
		line-height: 1.2;
		display: grid;
		grid-template-columns: 1fr 1fr;
	}
	.first {
		position: relative;
		/* height: calc(56px * 1.2); */
		/* overflow: hidden; */
	}
	.first > p {
		display: block;
		text-align: end;
	}

	.plane0 {
		width: 100%;
		height: 100%;
		grid-row: 2 / 7;
		grid-column: 2/7;
		align-self: start;
	}
	.plane1 {
		grid-row: 6 / 9;
		grid-column: 6/9;
	}
	.plane2 {
		width: 90%;
		height: 90%;
		justify-self: center;
		align-self: center;
		grid-row: 8 / 13;
		grid-column: 1/6;
	}
	.plane3 {
		width: 120%;
		height: 120%;
		grid-row: 10 / 12;
		grid-column: 5/7;
		justify-self: center;
		align-self: end;
	}
	.plane4 {
		width: 80%;
		height: 80%;
		grid-row: 13 / 18;
		grid-column: 1/6;
		justify-self: center;
		align-self: center;
	}
	.plane5 {
		width: 95%;
		height: 95%;
		grid-row: 13 / 18;
		grid-column: 6/11;
		justify-self: center;
		align-self: center;
	}
	.plane6 {
		grid-row: 18 / 23;
		grid-column: 5/10;
	}
	.plane7 {
		width: 70%;
		height: 70%;
		grid-row: 19 / 23;
		grid-column: 1/5;
		justify-self: center;
		align-self: start;
	}
	.plane8 {
		width: 90%;
		height: 90%;
		grid-row: 22 / 26;
		grid-column: 2/6;
		justify-self: center;
		align-self: center;
	}
	.plane9 {
		width: 100%;
		height: 100%;
		grid-row: 28 / 32;
		grid-column: 2/6;
		justify-self: center;
		align-self: center;
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
		height: auto;
		display: grid;
		justify-content: center;
		grid-template-columns: repeat(10, var(--column__width));
		grid-template-rows: 20px repeat(24, var(--row__height));
		/* background-color: #aeb4b8; */
		/* grid-gap: 2vw; */
	}
	.plane > picture > img {
		display: none;
		height: 100%;
		width: 100%;
		object-fit: cover;
		object-position: center;
	}
	@media (max-width: 800px) {
		.plane__wrapper {
			display: grid;
			justify-content: center;
			grid-template-columns: 1fr;
			grid-template-rows: auto;
		}
		.plane {
			width: 95vw;
			height: calc(95vw * 1.6);
		}
		.plane0 {
			grid-row: 2/3;
			grid-column: auto;
			align-self: start;
		}

		.plane1 {
			grid-row: 4/5;
			grid-column: auto;
			place-self: center;
		}
		.plane1,
		.plane3 {
			width: 70vw;
			height: calc(70vw * 1.6);
		}
		.plane2 {
			justify-self: center;
			align-self: center;
			grid-row: 6/7;
			grid-column: auto;
		}
		.plane3 {
			grid-row: 8/9;
			grid-column: auto;
			justify-self: center;
			align-self: center;
		}
		.plane4 {
			grid-row: 10/11;
			grid-column: auto;
			justify-self: center;
			align-self: center;
		}
		.plane5 {
			grid-row: auto;
			grid-column: auto;
			justify-self: center;
			align-self: center;
		}
		.plane6 {
			grid-row: auto;
			grid-column: auto;
			display: none;
		}
		.plane7 {
			grid-row: auto;
			grid-column: auto;
			justify-self: center;
			align-self: start;
			display: none;
		}
		.plane8 {
			grid-row: auto;
			grid-column: auto;
			justify-self: center;
			align-self: center;
			display: none;
		}
		.plane9 {
			grid-row: auto;
			grid-column: auto;
			justify-self: center;
			align-self: center;
			display: none;
		}
		.main__hedlain {
			grid-column: 1/2;
			grid-row: 1/2;
			text-align: center;
			margin-top: calc(2 * var(--vh));
		}
		.part1 {
			grid-row: 3/4;
			grid-column: auto;
			align-self: center;
			/* height: calc(30 * var(--vh)); */
			margin-top: calc(10 * var(--vh));
			margin-bottom: calc(10 * var(--vh));
			padding: calc(1 * var(--vh));
		}
		.part2 {
			grid-row: 5/6;
			grid-column: auto;
			padding: calc(1 * var(--vh));
			align-self: start;
			margin-bottom: calc(5 * var(--vh));
		}
		.part4 {
			grid-row: 7/8;
			grid-column: auto;
			margin: calc(5 * var(--vh)) 0;
			align-self: start;
			/* transform: translateY(-30%); */
		}
		.part4 > p {
			font-size: max(52px, 32px + 1.7vw);
		}
		.whom {
			grid-row: 9/10;
			grid-column: auto;
			transform: translateY(0%);
			margin: calc(5 * var(--vh)) 0;
		}
		.where {
			grid-row: 11/12;
			grid-column: auto;
			height: 100%;
			width: 100%;
			object-fit: cover;
			object-position: center;
			margin: calc(5 * var(--vh)) 0;
		}
		.date {
			font-size: 36px;
		}
	}
</style>
