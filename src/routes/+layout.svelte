<script lang="ts">
	// https://kit.svelte.dev/docs/routing#layout
	import '@shoelace-style/shoelace/dist/themes/light.css';
	import '@shoelace-style/shoelace/dist/themes/dark.css';
	import '../app.css';
	import { setBasePath } from '@shoelace-style/shoelace/dist/utilities/base-path.js';
	setBasePath('/node_modules/@shoelace-style/shoelace/dist');

	import { onMount } from 'svelte';
	import type { SlPopup, SlSwitch } from '@shoelace-style/shoelace/dist/shoelace.js';
	import Popover from '$lib/popover.svelte';
	import PopoverItem from '../lib/popover-item.svelte';
	import PopoverDivider from '../lib/popover-divider.svelte';

	// TODO: should support high-contrast in the future
	type Theme = 'light' | 'dark' | 'auto';

	// state
	let open = false;
	let theme = 'auto';
	let isDark = false;
	let online = true;

	// elements
	let popupElement: SlPopup;
	let rootElement: HTMLElement;

	// functionality
	function togglePopup(shouldOpen = !open) {
		open = shouldOpen;
		popupElement.active = open;
	}

	function setMode(selection: Theme = 'auto') {
		if (theme !== selection) {
			localStorage.setItem('theme', selection);
			theme = selection;
		}

		let prefersDark = selection === 'dark';
		if (selection === 'auto') {
			prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
		}
		isDark = prefersDark;

		rootElement.classList.toggle('sl-theme-dark', prefersDark);
	}

	// event handlers
	function onHandleTrigger(e: MouseEvent | KeyboardEvent) {
		// TODO: needs to support click outside
		if (
			(e as KeyboardEvent).key &&
			(e as KeyboardEvent).key !== 'Enter' &&
			(e as KeyboardEvent).key !== ' '
		) {
			return;
		}

		setMode(localStorage.getItem('theme') as Theme);
		togglePopup();
	}

	function onHandleOnlineChange(e: CustomEvent) {
		online = (e.target as SlSwitch).checked;
	}

	function onToggleModeClick() {
		setMode(isDark ? 'light' : 'dark');
	}

	onMount(async () => {
		// activates shoelace web components
		await import('@shoelace-style/shoelace/dist/shoelace.js');

		rootElement = document.documentElement;

		// sets the mode from preferences or local storage
		setMode();
	});

	// computed
	$: themeIcon = !isDark ? 'moon' : 'sun';
	$: themeLabel = !isDark ? 'Set Dark Mode' : 'Set Light Mode';
</script>

<div class="app">
	<sl-visually-hidden>
		<a href="#global-main" class="skip-link">Skip to main content</a>
	</sl-visually-hidden>
	<sl-visually-hidden>
		<a href="#global-nav" class="skip-link">Skip to navigation</a>
	</sl-visually-hidden>
	<header class="header">
		<div class="header__group">
			<sl-tooltip content="Menu">
				<sl-icon-button name="list" label="Menu" />
			</sl-tooltip>
			<sl-tooltip content="Search">
				<sl-icon-button name="search" label="Search" />
			</sl-tooltip>
		</div>
		<div class="header__group header__group--center">My App</div>
		<div class="header__group header__group--end">
			<sl-tooltip content="Call History">
				<sl-icon-button name="clock-history" label="Call History" />
			</sl-tooltip>
			<sl-popup bind:this={popupElement} placement="bottom" flip shift shift-padding="10">
				<sl-avatar
					on:click={onHandleTrigger}
					on:keydown={onHandleTrigger}
					tabindex="0"
					role="button"
					label="User avatar"
					slot="anchor"
					class="avatar {online ? 'is-online' : 'is-offline'}"
				/>
				<Popover>
					<PopoverItem class="user-info">
						<sl-icon slot="prefix" name="person" />
						<span>
							<span>User Name</span><br />
							<small>user.name@site.com</small>
						</span>
					</PopoverItem>
					<PopoverItem>
						<sl-switch size="medium" checked={online} on:sl-change={onHandleOnlineChange}
							>Online</sl-switch
						>
					</PopoverItem>
					<PopoverDivider />
					<PopoverItem>
						<sl-icon slot="prefix" name="receipt" />
						Orders
					</PopoverItem>
					<PopoverItem>
						<sl-icon slot="prefix" name="heart" />
						Wishlist
					</PopoverItem>
					<PopoverDivider />
					<PopoverItem tabindex={0}>
						<sl-icon slot="prefix" name={themeIcon} />
						{themeLabel}
					</PopoverItem>
					<PopoverItem tabindex={0}>
						<sl-icon slot="prefix" name="gear" />
						Settings
					</PopoverItem>
					<PopoverItem tabindex={0}>
						<sl-icon slot="prefix" name="box-arrow-right" />
						Logout
					</PopoverItem>
				</Popover>
			</sl-popup>
		</div>
	</header>
	<main id="global-main">
		<slot />
	</main>
	<nav id="global-nav" aria-label="Main Navigation">
		<a href="/">Home</a>
		<a href="/search">Search</a>
		<a href="/cart">Cart</a>
		<a href="/calls">Calls</a>
	</nav>
</div>

<style>
	.skip-link {
		position: absolute;
		top: 0;
		left: 0;
		background: var(--sl-panel-background-color);
		color: var(--sl-color-gray-700);
		padding: 0.5rem 1rem;
		border-radius: var(--sl-border-radius-medium);
		z-index: 100;
	}

	.header {
		display: flex;
		flex-direction: row;
		align-items: center;
		gap: var(--sl-spacing-x-small);
	}

	.header__group {
		display: inline-flex;
		flex-direction: row;
		align-items: center;
		gap: var(--sl-spacing-x-small);
	}

	.header__group--center {
		position: absolute;
		left: 50%;
		transform: translateX(-50%);
	}

	.header__group--end {
		margin-inline-start: auto;
	}

	.avatar {
		--size: var(--sl-font-size-2x-large);
		cursor: pointer;
		border-radius: var(--sl-border-radius-circle);
	}

	.avatar.is-offline {
		box-shadow: 0 0 0 2px var(--sl-color-danger-500);
	}

	.avatar.is-online {
		box-shadow: 0 0 0 2px var(--sl-color-success-500);
	}

	.user-info {
		--popover-label-line-height: var(--sl-line-height-denser);
	}
</style>
