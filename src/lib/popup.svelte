<script lang="ts">
	import { onDestroy, onMount } from 'svelte';

	let open = false;

	function onClick() {
		open = !open;
	}

	function onFocus(e: FocusEvent) {
		console.log('focus', e);
		// open = true;
	}

	function openPopupOnFocus(node: HTMLElement) {
		node.addEventListener('focus', onFocus, false);
		console.log('openPopupOnFocus', node);

		return {
			destroy() {
				node.removeEventListener('focus', onFocus);
			}
		};
	}
</script>

<sl-popup placement="bottom" active={open}>
	<slot slot="anchor" />
	<slot name="content" />
</sl-popup>
