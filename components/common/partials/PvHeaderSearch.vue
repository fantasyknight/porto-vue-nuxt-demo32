<template>
	<div
		class="header-icon header-search header-search-popup header-search-category d-none d-sm-block"
		@click.stop="toggleSearchForm($event)"
	>
		<a
			href="javascript:;"
			class="search-toggle"
			role="button"
			@click.stop="toggleSearchForm($event)"
		>
			<i class="icon-magnifier"></i>
		</a>

		<form
			action="#"
			method="get"
			@click.stop="toggleSearchForm($event)"
			@submit.prevent="submitSearchForm"
		>
			<div class="header-search-wrapper">
				<label
					for="search_term"
					class="sr-only"
				>Search</label>
				<input
					type="text"
					class="form-control"
					name="search_term"
					id="search_term"
					placeholder="I'm searching for..."
					required
					v-model="search_term"
					@input="searchProducts"
					@click.stop="showSearchForm($event)"
				/>
				<div
					class="select-custom"
					@click.stop="showSearchForm($event)"
				>
					<select
						id="cat"
						name="searchCategory"
						v-model="searchCategory"
						@change="searchProducts"
					>
						<option value>All Categories</option>
						<option value="fashion">Fashion</option>
						<option value="women">- Women</option>
						<option value="men">- Men</option>
						<option value="jewellery">- Jewellery</option>
						<option value="kids">- Kids Fashion</option>
						<option value="electronics">Electronics</option>
						<option value="smart-tvs">- Smart TVs</option>
						<option value="cameras">- Cameras</option>
						<option value="games">- Games</option>
						<option value="home-garden">Home &amp; Garden</option>
						<option value="motors">Motors</option>
						<option value="cars-and-trucks">- Cars and Trucks</option>
						<option value="motorcycles-powersports">
							- Motorcycles &amp;
							Powersports
						</option>
						<option value="accessories">
							- Parts &amp;
							Accessories
						</option>
						<option value="boats">- Boats</option>
						<option value="supplies">
							- Auto Tools &amp;
							Supplies
						</option>
					</select>
				</div>
				<button
					class="btn icon-magnifier"
					title="search"
					type="submit"
				></button>

				<div class="live-search-list">
					<div
						class="search-suggests"
						v-if="suggestions.length > 0"
						@click.prevent="goShopPage"
					>
						<nuxt-link
							:to="'/product/default/' + product.slug"
							class="search-suggest"
							data-index="0"
							v-for="product in suggestions"
							:key="product.id"
						>
							<div class="search-media">
								<img
									:src="`${baseUrl}${product.small_pictures[0].url}`"
									alt="Product"
									width="40"
									height="40"
									class="product-image"
								/>

								<div
									class="search-name"
									v-html="emphasizeMatchWord(product.name)"
								></div>
							</div>

							<div class="search-price">
								<div
									class="product-price mb-0"
									v-if="product.minPrice == product.maxPrice"
								>${{ product.minPrice | priceFormat }}</div>

								<template v-else>
									<div
										class="product-price mb-0"
										v-if="product.variants.length == 0"
									>
										<span class="new-price">${{ product.minPrice | priceFormat }}</span>
										<span class="old-price">${{ product.maxPrice | priceFormat }}</span>
									</div>
									<div
										class="product-price mb-0"
										v-else
									>${{product.minPrice | priceFormat}} - ${{product.maxPrice | priceFormat}}</div>
								</template>
							</div>
						</nuxt-link>
					</div>
				</div>
			</div>
		</form>
	</div>
</template>
<script>
import { mapGetters } from 'vuex';
import Api, { baseUrl, currentDemo } from '~/api';

export default {
	data: function () {
		return {
			search_term: '',
			suggestions: [],
			timeouts: [],
			baseUrl: baseUrl,
			currentDemo: currentDemo,
			searchCategory: ''
		};
	},
	mounted: function () {
		document
			.querySelector( 'body' )
			.addEventListener( 'click', this.closeSearchForm );
	},
	methods: {
		searchProducts: function () {
			if ( this.search_term.length > 2 ) {
				let search_term = this.search_term;
				this.timeouts.map( timeout => {
					window.clearTimeout( timeout );
				} );
				this.timeouts.push(
					setTimeout( () => {
						Api.get( `${ baseUrl }/search`, {
							params: {
								search_term: search_term,
								demo: this.currentDemo,
								category: this.searchCategory
							}
						} )
							.then( response => {
								this.suggestions = response.data.reduce(
									( acc, cur ) => {
										let max = 0;
										let min = 99999;
										min = cur.sale_price
											? cur.sale_price
											: 99999;
										max = cur.price ? cur.price : 0;
										if ( cur.variants && !cur.price ) {
											min = cur.variants[ 0 ].price;

											cur.variants.forEach( item => {
												let itemPrice = item.sale_price
													? item.sale_price
													: item.price;
												if ( min > itemPrice )
													min = itemPrice;
												if ( max < itemPrice )
													max = itemPrice;
											} );
										}
										if ( min === 99999 ) min = max;
										else if ( max === 0 ) max = min;
										return [
											...acc,
											{
												...cur,
												minPrice: min,
												maxPrice: max
											}
										];
									},
									[]
								);
							} )
							.catch( error => { } );
					}, 500 )
				);
			} else {
				this.timeouts.map( timeout => {
					window.clearTimeout( timeout );
				} );
				this.suggestions = [];
			}
		},
		emphasizeMatchWord: function ( name ) {
			let regExp = new RegExp( this.search_term, 'i' );
			return name.replace(
				regExp,
				match => '<strong>' + match + '</strong>'
			);
		},
		goShopPage: function () {
			this.search_term = '';
			this.suggestions = [];
		},
		searchToggle: function ( e ) {
			document.querySelector( '.header-search' ).classList.toggle( 'show' );
			e.stopPropagation();
		},
		toggleSearchForm: function ( e ) {
			e.currentTarget.closest( '.header-search' ).classList.toggle( 'show' );
		},
		showSearchForm: function ( e ) {
			e.currentTarget.closest( '.header-search' ).classList.add( 'show' );
		},
		closeSearchForm: function ( e ) {
			document
				.querySelector( '.header .header-search' )
				.classList.remove( 'show' );
		},
		submitSearchForm: function ( e ) {
			this.closeSearchForm();
			this.$router.push( {
				path: '/shop',
				query: {
					search_term: this.search_term,
					category: this.searchCategory
				}
			} );
		},
		handler( e ) {
			e.target.parentNode.setAttribute( 'style', 'display: none' );
		}
	}
};
</script>