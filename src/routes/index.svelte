<script>
	import { Curtains, TextureLoader, Plane, Vec3 } from 'curtainsjs/src/index.mjs';
	import { onMount } from 'svelte';
	import anime from 'animejs';
	import vertex from '../lib/photoday.vert?raw';
	import fragment from '../lib/photoday.frag?raw';
	let canvas,
		curtains,
		scrollEffect = 0,
		noiseFreq = 0.9,
		noiseAmp = 0.04,
		noiseEffect = 0,
		planes = [],
		animate,
		trans = 0,
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
			'2560_21-01-04-12-23-33',
			'20-09-06-13-12-33',
			'20-09-06-13-24-23',
			'20-09-06-13-22-33',
			'20-09-06-13-25-23',
			'20-08-31-12-24-31',
			'20-08-31-12-33-54',
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
			.add({
				targets: '.top',
				translateY: '-101%',
				easing: 'linear',
				duration: 500
			})
			.add(
				{
					targets: '.bottom',
					translateY: '101%',
					easing: 'linear',
					duration: 500
				},
				0
			)
			.add(
				{
					targets: '.webgl',
					scale: [1.13, 1],
					easing: 'linear',
					duration: 700
				},
				'-=300'
			)
			.add(
				{
					update: () => {
						scrollEffect += (80 - scrollEffect) * 0.3;
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
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
	<link
		href="https://fonts.googleapis.com/css2?family=Cormorant+Infant:wght@300&display=swap"
		rel="stylesheet"
	/>
</svelte:head>
<main>
	<div bind:this={canvas} class="webgl" />
	<div class="top half">
		<h2>Svobodina</h2>
	</div>
	<div class="bottom half">
		<h2>Photo</h2>
	</div>
	<div class="plane__wrapper">
		<h1 class="main__hedlain">Фотопроект<br />"Ничего лишнего"</h1>
		<p class="description part1">
			Фотографии на белом фоне, где главный герой "Вы", а не дизайн фотостудии
		</p>
		<p class="description part2">Только ваши эмоции и чувства</p>
		<div class="description part4">
			<p class="date">Дата</p>
			<p class="date">03.10.2021г.</p>
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
			<h2 class="description date">Контакты</h2>
			<div class="animate__contact">
				<div class="first">
					<p style="transform: translateY(calc(56px * -1.2 * {trans}));">instagram.com/</p>
					<p style="transform: translateY(calc(56px * -1.2 * {trans}));">vk.com/</p>
					<p style="transform: translateY(calc(56px * -1.2 * {trans}));">t.me/</p>
				</div>
				<div class="second"><p>svobodinaphoto</p></div>
			</div>
		</div>
		{#each photos as photo, index (index)}
			<div class="plane plane{index}">
				<img
					data-sampler="planeTexture"
					alt="SvobodinaPhoto"
					crossorigin="anonimous"
					decoding="async"
					draggable="false"
					src="./photoday/{photo}.webp"
				/>
			</div>
		{/each}
	</div>
</main>

<style>
	:global(body) {
		margin: 0;
	}
	:global(html) {
		font-family: 'Cormorant Infant', serif;
	}
	:root {
		--row__height: calc(var(--column__width) * 1.5);
		--column__width: min(8.5vw, 140px);
	}
	h2 {
		/* color: white; */
		font-size: max(32px, 8vw);
		font-family: 'Cormorant Infant', serif;
		font-weight: 300;
		margin: 0;
		text-align: center;
	}
	p {
		margin: 0;
	}
	.half {
		background-color: black;
		width: 100%;
		height: 50vh;
		position: fixed;
		display: grid;
		grid-template-columns: 1fr 1fr;
	}
	/* h2 {
		font-family: 'Roboto Slab', serif;
	} */
	.bottom h2 {
		grid-column: 2/3;
	}
	.top h2 {
		justify-self: end;
		align-self: end;
		grid-column: 1/2;
	}
	.bottom {
		top: 50vh;
	}
	.main__hedlain {
		margin: 0;
		z-index: 2;
		grid-row: 3 / 4;
		grid-column: 6/11;
		font-size: max(24px, 5vw);
		font-family: 'Cormorant Infant', serif;
		font-weight: 300;
	}
	.description {
		font-size: max(24px, 22px + 1.5vw);
		font-family: 'Cormorant Infant', serif;
		margin: 0;
	}
	.description2 {
		font-size: max(24px, 22px + 1vw);
		font-family: 'Cormorant Infant', serif;
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
		margin-top: 3vh;
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
		font-size: max(32px, 32px + 3vw);
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
		grid-template-columns: repeat(auto-fill, minmax(max(20vw, 250px), 1fr));
		row-gap: 5vh;
		column-gap: 2vw;
		width: 100%;
	}

	.price__tarif {
		position: relative;
		display: grid;
		grid-template-rows: 10% min(30vh 600px) 10%;
		border-radius: 5px;
		padding: 3vh 1vw;
	}
	.price__tarif > h3 {
		font-size: 36px;
		align-self: start;
		justify-self: center;
		white-space: pre-wrap;
		text-align: center;
	}
	.price__tarif > ul {
		align-self: stretch;
		margin-top: 5vh;
		margin-bottom: 5vh;
	}
	li {
		font-size: 26px;
		font-weight: 400;
		/* list-style-type: '📷'; */
		padding-inline-start: 1ch;
	}
	.price__tarif > h4 {
		font-size: 36px;
		place-self: center;
		white-space: pre-wrap;
	}
	.contact {
		grid-row: 30 / 32;
		grid-column: 1/11;
		height: 100%;
		width: 100%;
	}
	.animate__contact {
		font-size: 56px;
		line-height: 1.2;
		display: grid;
		grid-template-columns: 1fr 1fr;
	}
	.first {
		height: calc(56px * 1.2);
		overflow: hidden;
	}
	.first > p {
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
		/* width: 90%;
  height: 90%; */
		grid-row: 18 / 23;
		grid-column: 5/10;
		/* justify-self: center;
  align-self: center; */
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
		grid-template-rows: 20px repeat(34, var(--row__height));
		/* grid-gap: 2vw; */
	}
	.plane > img {
		display: none;
		height: 100%;
		width: 100%;
		object-fit: cover;
		object-position: center;
	}
</style>
