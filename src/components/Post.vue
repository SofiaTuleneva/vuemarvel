<template lang="html">
	<div class="post" v-if="post">
		<div>
			<h1 class="post__title">{{ post.name }}</h1>
			<p class="post__img">
				<img v-bind:src="post.thumbnail.path + '.' + post.thumbnail.extension" />
			</p>
			<p class="post__body">{{ post.description }}</p>
			<p class="post__id">{{ post.id }}</p>
		</div>
		<div v-if="state.loading">
			<preloader></preloader>
		</div>
	</div>
</template>

<script>
	import axios from 'axios';
	import Preloader from './Preloader.vue';
	
	export default {
		props: ['id'],
		
		data() {
			return {
				post: null,
				endpoint: 'https://gateway.marvel.com/v1/public/characters',
				state: {
					loading: false
				},
			}
		},
		
		components: {
			Preloader
		},

		created() {
			this.getPost(this.id);
		},

		methods: {
			getPost(id) {
				this.state.loading = true;
				
				axios(this.endpoint + '/' + id, {
					params: {
						apikey: '05b2be7446add8d4fc42d9c44ee1b846',
						ts: 1
					}
				})
				.then(response => {
					this.post = response.data.data.results[0]
				})
				.catch( error => {
					console.log(error)
				})
				.then(() => {
					this.state.loading = false;
				})
			}
		},

		watch: {
			'$route'() {
				this.getPost(this.id);
			}
		},
	}
</script>

<style lang="scss" scoped>
	.post {
		position: relative;
		max-width: 500px;
		margin: 0 auto;
		padding: 50px 20px 70px;
		text-align: center;
		&__title {
			position: relative;
			text-transform: uppercase;
			z-index: 1;
		}
		&__body {
			position: relative;
			z-index: 1;
		}
		&__id {
			font-size: 100px;
			margin: 0;
			color: #eeeeee;
			line-height: 1;
			font-weight: 900;
			z-index: 0;
		}
		&__img {
			img {
				width: 50%;
			}
		 }
	}
</style>