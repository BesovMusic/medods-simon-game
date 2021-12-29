<template>
	<SimonBoard />
</template>

<script>
import SimonBoard from './components/SimonBoard.vue';

export default {
	name: 'App',
	components: {
		SimonBoard,
	},
};
</script>

<style lang="scss">
* {
	box-sizing: border-box;
}
body {
	margin: 0;
}
</style>
