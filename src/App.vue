<template>
	<div id="app">
		<header>
			<h1>
				<div id="mvl-nav-top-c1">
					<a href="/" class="icon-marvel-logo-svg" title="marvel" alt="marvel"></a>
				</div>
			</h1>
			<ul>
				<li v-for="char in alphabet">
					<span v-on:click="getPostsByNameStartsWith(char, true)" class="link-text">{{ char }}</span>
				</li>
			</ul>
		</header>
		<main>
			<aside class="sidebar">
				<div>
					<input v-model="search" v-on:keyup="getPostsByNameStartsWith(search)" class="search" placeholder="Character name"/>
				</div>
				<p>
					<button v-on:click="getPostsPrev()" :disabled="!state.prev">Prev</button>
					<button v-on:click="getPostsNext()" :disabled="!state.next">Next</button>
				</p>
				<router-link
						v-for="post in posts"
						active-class="is-active"
						class="link"
						:to="{ name: 'post', params: { id: post.id } }">
					{{post.name}}
				</router-link>
				<p v-if="state.next">
					<button v-on:click="getPostsPrev()" :disabled="!state.prev">Prev</button>
					<button v-on:click="getPostsNext()" :disabled="!state.next">Next</button>
				</p>
			</aside>
			<div class="content">
				<router-view></router-view>
			</div>
		</main>
		<div v-if="state.loading">
			<preloader></preloader>
		</div>
	</div>
</template>

<script>
	import axios from 'axios';
	import Preloader from './components/Preloader.vue';

	export default {
		data() {
			return {
				posts: [],
				alphabet: [],
				endpoint: 'https://gateway.marvel.com/v1/public/characters',
				search: '',
				limit: 50,
				state: {
					offset: 0,
					nameStartsWith: "",
					next: true,
					prev: true,
					loading: false,
					cancelSource: {},
				},
			}
		},
		
		components: {
			Preloader
		},
		
		created() {
			this.alphabet = genCharArray('a','z');
			this.getPosts(this.state.offset, this.state.nameStartsWith);
		},
		
		methods: {
			getPosts(offset, nameStartsWith) {
				
				if (nameStartsWith) {
					this.state.nameStartsWith = nameStartsWith;
				} else if (nameStartsWith === '') {
					this.state.nameStartsWith = undefined;
				}

				const CancelToken = axios.CancelToken;
				this.state.cancelSource = CancelToken.source();
				
				this.state.loading = true;

//				// Add a request interceptor
//				axios.interceptors.request.use(function (config) {
//					// Do something before request is sent
//					self.state.loading = true;
//					alert('start loading');
//					return config;
//				}, function (error) {
//					// Do something with request error
//					return Promise.reject(error);
//				});
				
				axios.get(this.endpoint, {
					cancelToken: this.state.cancelSource.token,
					params: {
						apikey: '05b2be7446add8d4fc42d9c44ee1b846',
						ts: 1,
						limit: this.limit,
						offset: offset,
						nameStartsWith: this.state.nameStartsWith,
					}
				})
				.then(response => {
					let data = response.data.data;
					
					// save posts
					this.posts = data.results;
					
					// set state
					this.state.offset = data.offset;
					this.state.next = (data.total - data.offset) > this.limit;
					this.state.prev = data.offset > 0;

					console.log(data, 'nameStartsWith:' + this.state.nameStartsWith);
				})
				.catch(thrown => {
					if (axios.isCancel(thrown)) {
						// handle cancel
						console.log('Request canceled', thrown.message);
					} else {
						// handle error
						this.state.loading = false;
						console.log('-----error-------', thrown);
					}
				})
				.then(() => {
					this.state.loading = false;
				})
			},
		
			getPostsNext() {
				this.getPosts(this.state.offset + this.limit);
			},
			
			getPostsPrev() {
				this.getPosts(this.state.offset - this.limit);
			},
			
			getPostsByNameStartsWith(nameStartsWith, clearSearch) {
				if (clearSearch) this.search = '';
				this.state.cancelSource.cancel(nameStartsWith);
				this.getPosts(0, nameStartsWith);
			},
		}
	}
	
	function genCharArray(charA, charZ) {
		var a = [], i = charA.charCodeAt(0), j = charZ.charCodeAt(0);
		for (; i <= j; ++i) {
			a.push(String.fromCharCode(i));
		}
		return a;
	}
</script>

<style lang="scss">
	body {
		margin: 0;
		padding: 0;
	}
	
	#app {
		font-family: 'Avenir', Helvetica, Arial, sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
		color: #2c3e50;
	}
	
	h1, h2 {
		font-weight: normal;
	}
	
	ul {
		list-style-type: none;
		padding: 0;
	}
	
	li {
		display: inline-block;
		margin: 0 10px;
	}
	
	.link-text {
		text-decoration: underline;
		cursor: pointer;
	}
	
	header {
		/*position: fixed;*/
		top: 0;
		width: 100%;
		/*min-height: 90px;*/
		border-bottom: 1px solid #42b983;
		text-align: center;
		background: #ffffff;
	}
	
	main {
		display: flex;
		height: calc(100vh - 90px);
		max-width: 1200px;
		/*margin-top: 90px;*/
		margin-left: auto;
		margin-right: auto;
		overflow: hidden;
	}
	
	aside {
		flex: 1 0 30%;
		height: 100%;
		overflow-y: auto;
		width: 30%;
		padding: 50px 30px;
		box-sizing: border-box;
		border-right: 1px solid #42b983;
	}
	
	.content {
		flex: 1 1 70%;
		display: flex;
		align-items: center;
		justify-content: center;
	}
	
	.link {
		display: block;
		text-decoration: none;
		margin-bottom: 10px;
		color: #2c3e50;
		&--home {
			text-transform: uppercase;
			margin-bottom: 30px;
		}
		&.is-active {
			color: #42b983;
		}
	}
	
	#mvl-nav-top-c1 {
		display: inline-block;
		margin: 0 auto;
		padding-top: 3px;
		padding-left: 3px;
		padding-right: 3px;
	}
	
	.icon-marvel-logo-svg {
		display: block;
		width: 200px;
		height: 78px;
		background: url(assets/marvel.svg);
		background-size: 200px 78px;
	}
	
	.search {
		display: inline-block;
		width: 100%;
	}
</style>