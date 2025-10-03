<script lang="ts">
	import { onMount } from 'svelte';
	import { fade, fly, scale } from 'svelte/transition';
	import { elasticOut, quintOut } from 'svelte/easing';

	const targetDate = new Date('2025-10-24T00:00:00Z');
	const loadingTexts = [
		'"I ain\'t got time for this shit..."',
		'"Another f***ing loading screen..."'
	];

	let timeLeft = $state({ days: 0, hours: 0, minutes: 0, seconds: 0 });
	let videoLoaded = $state(false);
	let contentVisible = $state(false);
	let randomLoadingText = $state('');
	let loadedVideo: HTMLVideoElement;
	let hiddenVideo: HTMLVideoElement;

	let interval: ReturnType<typeof setInterval> | undefined;

	// Google Analytics tracking functions
	function trackPresaveClick() {
		if (typeof window !== 'undefined' && (window as any).gtag) {
			(window as any).gtag('event', 'click', {
				event_category: 'engagement',
				event_label: 'presave_button',
				value: 1
			});
		}
	}

	function trackSocialClick(platform: string, url: string) {
		if (typeof window !== 'undefined' && (window as any).gtag) {
			(window as any).gtag('event', 'click', {
				event_category: 'social_media',
				event_label: platform.toLowerCase(),
				value: 1
			});
		}
	}

	function updateCountdown() {
		const now = new Date();
		const diff = targetDate.getTime() - now.getTime();

		if (diff <= 0) {
			timeLeft = { days: 0, hours: 0, minutes: 0, seconds: 0 };
			if (interval) clearInterval(interval);
			return;
		}

		const days = Math.floor(diff / (1000 * 60 * 60 * 24));
		const hours = Math.floor((diff / (1000 * 60 * 60)) % 24);
		const minutes = Math.floor((diff / (1000 * 60)) % 60);
		const seconds = Math.floor((diff / 1000) % 60);

		timeLeft = { days, hours, minutes, seconds };
	}

	function handleVideoLoad() {
		console.log('Video loaded!');
		videoLoaded = true;
		
		// Try to play the hidden video first
		if (hiddenVideo) {
			tryPlayVideo(hiddenVideo);
		}
		
		// Delay content appearance for smooth transition
		setTimeout(() => {
			contentVisible = true;
			// Try to play the visible video too
			if (loadedVideo) {
				tryPlayVideo(loadedVideo);
			}
		}, 300);
	}

	function tryPlayVideo(videoElement: HTMLVideoElement) {
		// Attempt to play video programmatically
		const playPromise = videoElement.play();
		
		if (playPromise !== undefined) {
			playPromise
				.then(() => {
					console.log('Video autoplay successful');
				})
				.catch((error) => {
					console.log('Video autoplay failed, but that\'s ok on mobile:', error);
					// Video will remain muted and loop, just won't autoplay
				});
		}
	}

	// Fallback timeout in case video never loads
	function startFallbackTimer() {
		setTimeout(() => {
			if (!videoLoaded) {
				console.log('Video loading timeout, showing content anyway');
				videoLoaded = true;
				setTimeout(() => {
					contentVisible = true;
				}, 300);
			}
		}, 1000);
	}

	onMount(() => {
		// Pick a random loading text
		randomLoadingText = loadingTexts[Math.floor(Math.random() * loadingTexts.length)];

		updateCountdown();
		interval = setInterval(updateCountdown, 1000);
		startFallbackTimer();
		return () => {
			if (interval) clearInterval(interval);
		};
	});
</script>

<!-- Main container to prevent any scrolling -->
<div class="fixed inset-0 h-[100dvh] w-screen overflow-hidden">
	<!-- Loading Screen -->
	{#if !videoLoaded}
		<div
			class="absolute inset-0 z-50 flex items-center justify-center bg-black"
			in:fade={{ duration: 0 }}
			out:fade={{ duration: 800, easing: quintOut }}
		>
			<div class="flex flex-col items-center gap-4">
				<!-- Loading Spinner -->
				<div
					class="h-12 w-12 animate-spin rounded-full border-4 border-white/20 border-t-white"
					in:scale={{ duration: 600, easing: elasticOut }}
				></div>
				<!-- Loading Text -->
				<p
					class="text-sm tracking-wide text-white/80"
					in:fly={{ y: 20, duration: 600, delay: 200 }}
				>
					{randomLoadingText}
				</p>
			</div>
		</div>
	{/if}

	<!-- Video Background -->
	{#if videoLoaded}
		<video
			class="absolute h-[100dvh] w-full origin-center scale-110 object-cover brightness-90 saturate-[1.25]"
			src="/img/canvas.MP4"
			autoplay
			muted
			loop
			playsinline
			controls={false}
			disablepictureinpicture
			controlslist="nodownload nofullscreen noremoteplayback"
			bind:this={loadedVideo}
			in:fade={{ duration: 1200, easing: quintOut }}
		></video>
	{:else}
		<video
			class="absolute h-[100dvh] w-full origin-center scale-110 object-cover opacity-0 brightness-90 saturate-[1.25]"
			src="/img/canvas.MP4"
			autoplay
			muted
			loop
			playsinline
			controls={false}
			disablepictureinpicture
			controlslist="nodownload nofullscreen noremoteplayback"
			onloadeddata={handleVideoLoad}
			oncanplay={handleVideoLoad}
			onloadedmetadata={handleVideoLoad}
			bind:this={hiddenVideo}
		></video>
	{/if}

	<!-- Content Layer -->
	{#if contentVisible}
		<!-- Logo -->
		<img
			src="/img/logo.png"
			alt="Maybon Logo"
			class="absolute top-20 left-1/2 mx-auto w-3/4 max-w-md -translate-x-1/2"
			in:fly={{ y: -30, duration: 800, delay: 200, easing: quintOut }}
		/>

		<!-- Main Content -->
		<div
			class="absolute top-0 left-0 flex h-full w-full flex-col items-center justify-center !text-white"
			in:fade={{ duration: 600, delay: 400 }}
		>
			<!-- Countdown -->
			<div
				class="mt-4 flex gap-4 text-4xl text-white"
				style="font-family: 'Shadows Into Light Two', cursive;"
				in:fly={{ y: 20, duration: 800, delay: 600, easing: quintOut }}
			>
				{#each [{ value: timeLeft.days, suffix: 'd' }, { value: timeLeft.hours, suffix: 'h' }, { value: timeLeft.minutes, suffix: 'm' }, { value: timeLeft.seconds, suffix: 's' }] as item, i}
					<span
						in:scale={{
							duration: 600,
							delay: 800 + i * 100,
							easing: elasticOut
						}}
					>
						{item.value}{item.suffix}
					</span>
				{/each}
			</div>

			<!-- Presave Button -->
			<a
				href="https://distrokid.com/hyperfollow/maybon/no-friends?utm_campaign=website&utm_medium=Email+&utm_source=SendGrid"
				target="_blank"
				rel="noopener noreferrer"
				class="mt-2 rounded-full px-8 py-3 text-lg font-extralight tracking-[3px] text-white uppercase underline transition-all hover:scale-105"
				in:fly={{ y: 30, duration: 800, delay: 1200, easing: quintOut }}
				onclick={trackPresaveClick}
			>
				presave now
			</a>
		</div>

		<!-- Social Icons -->
		<div
			class="absolute bottom-20 left-1/2 grid -translate-x-1/2 grid-cols-3 gap-x-12 gap-y-6"
			in:fly={{ y: 30, duration: 800, delay: 1000, easing: quintOut }}
		>
			{#each [{ href: 'https://open.spotify.com/artist/58WNaQYHzXvOY23UGICpOb?si=1MxPTnLbSAebTJ5mjRAZqQ', label: 'Spotify', hoverColor: 'hover:fill-green-500', path: 'M12 0C5.4 0 0 5.4 0 12s5.4 12 12 12 12-5.4 12-12S18.66 0 12 0zm5.521 17.34c-.24.359-.66.48-1.021.24-2.82-1.74-6.36-2.101-10.561-1.141-.418.122-.859-.179-.982-.599-.122-.421.18-.861.599-.983 4.56-1.021 8.52-.621 11.757 1.818.359.179.48.66.208 1.665zm1.44-3.3c-.301.42-.841.6-1.262.3-3.239-1.98-8.159-2.58-11.939-1.38-.479.12-1.02-.12-1.14-.6-.12-.48.12-1.021.6-1.141C9.6 9.9 15 10.561 18.72 12.84c.361.181.54.78.241 1.2zm.12-3.36C15.24 8.4 8.82 8.16 5.16 9.301c-.6.179-1.2-.181-1.38-.721-.18-.601.18-1.2.72-1.381 4.26-1.32 11.28-1.08 15.721 1.621.539.3.719 1.02.419 1.56-.299.421-1.02.599-1.559.3z' }, { href: 'https://music.apple.com/no/artist/maybon/1048212431', label: 'Apple Music', hoverColor: 'hover:fill-gray-300', path: 'M23.997 6.124c0-.738-.065-1.47-.24-2.19-.317-1.31-1.062-2.31-2.18-3.043C21.003.517 20.373.285 19.7.164c-.517-.093-1.038-.135-1.564-.15-.04-.003-.083-.01-.124-.013H5.988c-.152.01-.303.017-.455.026C4.786.07 4.043.15 3.34.428 2.004.958 1.04 1.88.475 3.208c-.192.448-.292.925-.363 1.408-.056.392-.088.785-.1 1.18 0 .032-.007.062-.01.093v12.223c.01.14.017.283.027.424.05.815.154 1.624.497 2.373.65 1.42 1.738 2.353 3.234 2.802.42.127.856.187 1.293.228.555.053 1.11.06 1.667.06h11.03c.525 0 1.048-.034 1.57-.1.823-.106 1.597-.35 2.296-.81.84-.553 1.472-1.287 1.88-2.208.186-.42.293-.87.37-1.324.113-.675.138-1.358.137-2.04-.002-3.8 0-7.595-.003-11.393zm-6.423 3.99v5.712c0 .417-.058.827-.244 1.206-.29.59-.76.962-1.388 1.14-.35.1-.706.157-1.07.173-.95.045-1.773-.6-1.943-1.536-.142-.773.227-1.624 1.038-2.022.323-.16.67-.25 1.018-.324.378-.082.758-.153 1.134-.24.274-.063.457-.23.51-.516.014-.063.02-.13.02-.193 0-1.815 0-3.63-.002-5.443 0-.062-.01-.125-.026-.185-.04-.15-.15-.243-.304-.234-.16.01-.318.035-.475.066-.76.15-1.52.303-2.28.456l-2.326.47-1.374.278c-.016.003-.032.01-.048.013-.277.077-.377.203-.39.49-.002.042 0 .086 0 .13-.002 2.602 0 5.204-.003 7.805 0 .42-.047.836-.215 1.227-.278.64-.77 1.04-1.434 1.233-.35.1-.71.16-1.075.172-.96.036-1.755-.6-1.92-1.544-.14-.812.23-1.685 1.154-2.075.357-.15.73-.232 1.108-.31.287-.06.575-.116.86-.177.383-.083.583-.323.6-.714v-.15c0-2.96 0-5.922.002-8.882 0-.123.013-.25.042-.37.07-.285.273-.448.546-.518.255-.066.515-.112.774-.165.733-.15 1.466-.296 2.2-.444l2.27-.46c.67-.134 1.34-.27 2.01-.403.22-.043.443-.088.664-.106.31-.025.523.17.554.482.008.073.012.148.012.223.002 1.91.002 3.822 0 5.732z' }, { href: 'https://www.instagram.com/maybonmusic/', label: 'Instagram', hoverColor: 'hover:fill-pink-500', path: 'M12 2.163c3.204 0 3.584.012 4.85.07 3.252.148 4.771 1.691 4.919 4.919.058 1.265.069 1.645.069 4.849 0 3.205-.012 3.584-.069 4.849-.149 3.225-1.664 4.771-4.919 4.919-1.266.058-1.644.07-4.85.07-3.204 0-3.584-.012-4.849-.07-3.26-.149-4.771-1.699-4.919-4.92-.058-1.265-.07-1.644-.07-4.849 0-3.204.013-3.583.07-4.849.149-3.227 1.664-4.771 4.919-4.919 1.266-.057 1.645-.069 4.849-.069zm0-2.163c-3.259 0-3.667.014-4.947.072-4.358.2-6.78 2.618-6.98 6.98-.059 1.281-.073 1.689-.073 4.948 0 3.259.014 3.668.072 4.948.2 4.358 2.618 6.78 6.98 6.98 1.281.058 1.689.072 4.948.072 3.259 0 3.668-.014 4.948-.072 4.354-.2 6.782-2.618 6.979-6.98.059-1.28.073-1.689.073-4.948 0-3.259-.014-3.667-.072-4.947-.196-4.354-2.617-6.78-6.979-6.98-1.281-.059-1.69-.073-4.949-.073zm0 5.838c-3.403 0-6.162 2.759-6.162 6.162s2.759 6.163 6.162 6.163 6.162-2.759 6.162-6.163c0-3.403-2.759-6.162-6.162-6.162zm0 10.162c-2.209 0-4-1.79-4-4 0-2.209 1.791-4 4-4s4 1.791 4 4c0 2.21-1.791 4-4 4zm6.406-11.845c-.796 0-1.441.645-1.441 1.44s.645 1.44 1.441 1.44c.795 0 1.439-.645 1.439-1.44s-.644-1.44-1.439-1.44z' }, { href: 'https://www.tiktok.com/@maybonmusic', label: 'TikTok', hoverColor: 'hover:fill-red-500', path: 'M12.525.02c1.31-.02 2.61-.01 3.91-.02.08 1.53.63 3.09 1.75 4.17 1.12 1.11 2.7 1.62 4.24 1.79v4.03c-1.44-.05-2.89-.35-4.2-.97-.57-.26-1.1-.59-1.62-.93-.01 2.92.01 5.84-.02 8.75-.08 1.4-.54 2.79-1.35 3.94-1.31 1.92-3.58 3.17-5.91 3.21-1.43.08-2.86-.31-4.08-1.03-2.02-1.19-3.44-3.37-3.65-5.71-.02-.5-.03-1-.01-1.49.18-1.9 1.12-3.72 2.58-4.96 1.66-1.44 3.98-2.13 6.15-1.72.02 1.48-.04 2.96-.04 4.44-.99-.32-2.15-.23-3.02.37-.63.41-1.11 1.04-1.36 1.75-.21.51-.15 1.07-.14 1.61.24 1.64 1.82 3.02 3.5 2.87 1.12-.01 2.19-.66 2.77-1.61.19-.33.4-.67.41-1.06.1-1.79.06-3.57.07-5.36.01-4.03-.01-8.05.02-12.07z' }, { href: 'https://soundcloud.com/maybonofficial', label: 'Soundcloud', hoverColor: 'hover:fill-orange-500', path: 'M1.175 12.225c-.051 0-.094.046-.101.1l-.233 2.154.233 2.105c.007.058.05.098.101.098.05 0 .09-.04.099-.098l.255-2.105-.27-2.154c0-.057-.045-.1-.09-.1m-.899.828c-.06 0-.091.037-.104.094L0 14.479l.165 1.308c0 .055.045.094.09.094s.089-.045.104-.104l.21-1.319-.21-1.334c0-.061-.044-.09-.09-.09m1.83-1.229c-.061 0-.12.045-.12.104l-.21 2.563.225 2.458c0 .06.045.12.119.12.061 0 .105-.061.121-.12l.254-2.474-.254-2.548c-.016-.06-.061-.12-.121-.12m.945-.089c-.075 0-.135.06-.15.135l-.193 2.64.21 2.544c.016.077.075.138.149.138.075 0 .135-.061.15-.15l.24-2.532-.24-2.623c0-.075-.06-.135-.135-.135l-.031-.017zm1.155.36c-.005-.09-.075-.149-.159-.149-.09 0-.158.06-.164.149l-.217 2.43.2 2.563c0 .09.075.157.159.157.074 0 .148-.068.148-.158l.227-2.563-.227-2.444.033.015zm.809-1.709c-.101 0-.18.09-.18.181l-.21 3.957.187 2.563c0 .09.08.164.18.164.094 0 .174-.09.18-.18l.209-2.563-.209-3.972c-.008-.104-.088-.18-.18-.18m.959-.914c-.105 0-.195.09-.203.194l-.18 4.872.165 2.548c0 .12.09.209.195.209.104 0 .194-.089.21-.209l.193-2.548-.192-4.856c-.016-.12-.105-.21-.21-.21m.989-.449c-.121 0-.211.089-.225.209l-.165 5.275.165 2.52c.014.119.104.225.225.225.119 0 .225-.105.225-.225l.195-2.52-.196-5.275c0-.12-.105-.225-.225-.225m1.245.045c0-.135-.105-.24-.24-.24-.119 0-.24.105-.24.24l-.149 5.441.149 2.503c.016.135.121.24.256.24s.24-.105.24-.24l.164-2.503-.164-5.456-.016.015zm.749-.134c-.135 0-.255.119-.255.254l-.15 5.322.15 2.473c0 .15.12.255.255.255s.255-.12.255-.27l.15-2.474-.165-5.307c0-.148-.12-.27-.271-.27m1.005.166c-.164 0-.284.135-.284.285l-.103 5.143.135 2.474c0 .149.119.277.284.277.149 0 .271-.12.284-.285l.121-2.443-.135-5.112c-.012-.164-.135-.285-.285-.285m1.184-.945c-.045-.029-.105-.044-.165-.044s-.119.015-.165.044c-.09.054-.149.15-.149.255v.061l-.104 6.048.115 2.449v.008c.008.06.03.135.074.18.058.061.142.104.234.104.08 0 .158-.044.209-.09.058-.06.091-.135.091-.225l.015-.24.117-2.203-.135-6.086c0-.104-.061-.193-.135-.239l-.002-.022zm1.006-.547c-.045-.045-.09-.061-.15-.061-.074 0-.149.016-.209.061-.075.061-.119.15-.119.24v.029l-.137 6.609.076 1.215.061 1.185c0 .164.148.314.328.314.181 0 .33-.15.33-.329l.15-2.414-.15-6.637c0-.12-.074-.221-.165-.277m8.934 3.777c-.405 0-.795.086-1.139.232-.24-2.654-2.46-4.736-5.188-4.736-.659 0-1.305.135-1.889.359-.225.09-.27.18-.285.359v9.368c.016.18.15.33.33.345h8.185C22.681 17.218 24 15.914 24 14.28s-1.319-2.952-2.938-2.952' }, { href: 'https://www.facebook.com/Maybon.Official/', label: 'Facebook', hoverColor: 'hover:fill-blue-500', path: 'M24 12.073c0-6.627-5.373-12-12-12s-12 5.373-12 12c0 5.99 4.388 10.954 10.125 11.854v-8.385H7.078v-3.47h3.047V9.43c0-3.007 1.792-4.669 4.533-4.669 1.312 0 2.686.235 2.686.235v2.953H15.83c-1.491 0-1.956.925-1.956 1.874v2.25h3.328l-.532 3.47h-2.796v8.385C19.612 23.027 24 18.062 24 12.073z' }] as social, i}
				<a
					href={social.href}
					target="_blank"
					rel="noopener noreferrer"
					aria-label={social.label}
					in:scale={{
						duration: 500,
						delay: 1400 + i * 100,
						easing: elasticOut
					}}
					onclick={() => trackSocialClick(social.label, social.href)}
				>
					<svg
						class="h-10 w-10 fill-white transition-all duration-300 {social.hoverColor} hover:scale-110"
						viewBox="0 0 24 24"
					>
						<path d={social.path} />
					</svg>
				</a>
			{/each}
		</div>
	{/if}
</div>

<style>
	/* Hide video controls completely */
	video::-webkit-media-controls {
		display: none !important;
	}
	
	video::-webkit-media-controls-panel {
		display: none !important;
	}
	
	video::-webkit-media-controls-play-button {
		display: none !important;
	}
	
	video::-webkit-media-controls-start-playback-button {
		display: none !important;
	}
	
	/* For Firefox */
	video::-moz-media-controls {
		display: none !important;
	}
	
	/* General fallback */
	video {
		outline: none;
	}
	
	video:focus {
		outline: none;
	}
</style>
