<template>
	<main class="main skeleton-body product-layout-transparent mb-5">
		<div class="container-fluid">
			<div class="row">
				<div class="col-xl-2cols col-lg-3">
					<div
						class="sidebar-overlay"
						@click="hideSidebar"
					></div>
					<div
						class="sidebar-toggle custom-sidebar-toggle"
						@click="showSidebar"
					><i class="fas fa-sliders-h"></i></div>
					<aside
						class="sidebar-shop mobile-sidebar"
						sticky-container
					>
						<pv-sidebar-filter-one
							:category-list="categoryList"
							v-if="categoryList"
						></pv-sidebar-filter-one>

						<div
							class="sidebar-content skeleton-body"
							v-else
						>
							<aside class="widget"></aside>
							<aside class="widget"></aside>
							<aside class="widget"></aside>
						</div>
					</aside>
				</div>

				<div class="col-xl-8cols col-lg-9">
					<nav
						aria-label="breadcrumb"
						class="breadcrumb-nav"
					>
						<ol class="breadcrumb">
							<li class="breadcrumb-item">
								<nuxt-link to="/">
									Home
								</nuxt-link>
							</li>
							<li class="breadcrumb-item">
								<nuxt-link to="/shop">Shop</nuxt-link>
							</li>
							<li
								class="breadcrumb-item"
								v-if="loaded"
							>
								<nuxt-link
									:to="{path: '/shop', query: {category: category.slug}}"
									v-for="(category, index) in productCategory"
									:key="`product-category-${index}`"
								>{{index === productCategory.length - 1 ? category.name : category.name + ', '}}</nuxt-link>
							</li>
							<li
								class="breadcrumb-item active"
								aria-current="page"
								v-if="loaded"
							>{{product.name}}</li>
						</ol>
					</nav>

					<div v-if="!product">
						<div class="skel-group main-content">
							<div class="summary-before col-lg-7"></div>
							<div class="summary entry-summary col-lg-5"></div>
							<div class="tab-content col-lg-12"></div>
						</div>
					</div>

					<div class="product-single-container product-single-default product-transparent-image">
						<div
							class="row"
							v-if="product"
						>
							<div class="col-xl-7">
								<pv-media-five :product="product"></pv-media-five>
							</div>

							<div class="col-xl-5 product-single-details pt-3">
								<pv-detail-one
									:product="product"
									:prev-product="prevProduct"
									:next-product="nextProduct"
								></pv-detail-one>
							</div>
						</div>
					</div>

					<pv-desc-two
						:product="product"
						v-if="product"
					></pv-desc-two>

					<div class="mb-5"></div>

					<pv-related-products :products="relatedProducts"></pv-related-products>
				</div>

				<div class="col-xl-2cols col-12">
					<pv-product-sidebar-three></pv-product-sidebar-three>
				</div>
			</div>
		</div>
	</main>
</template>

<script>
import PvMediaFive from '~/components/partials/product/media/PvMediaFive';
import PvDetailOne from '~/components/partials/product/detail/PvDetailOne';
import PvDescTwo from '~/components/partials/product/description/PvDescTwo';
import PvRelatedProducts from '~/components/partials/product/PvRelatedProducts';
import Api, { baseUrl, currentDemo } from '~/api';
import PvSidebarFilterOne from '~/components/partials/shop/sidebar-filter/PvSidebarFilterOne';
import PvProductSidebarThree from '~/components/partials/product/sidebar/PvProductSidebarThree';
import PvSmallCollection from '~/components/partials/product/PvSmallCollection';

export default {
	components: {
		PvMediaFive,
		PvDetailOne,
		PvDescTwo,
		PvRelatedProducts,
		PvSmallCollection,
		PvSidebarFilterOne,
		PvProductSidebarThree
	},
	data: function () {
		return {
			product: null,
			relatedProducts: null,
			featuredProducts: null,
			bestProducts: null,
			latestProducts: null,
			topRatedProducts: null,
			nextProduct: null,
			prevProduct: null,
			loaded: false,
			productCategory: [],
			categoryList: []
		};
	},
	created: function () {
		this.getProduct();
		this.getCategoryLists();
	},
	methods: {
		getCategoryLists: function () {
			Api.get( `${ baseUrl }/shop/sidebar-list`, {
				params: { demo: currentDemo }
			} )
				.then( response => {
					this.categoryList = response.data.sidebarList;
				} )
				.catch( error => ( { error: JSON.stringify( error ) } ) );
		},
		getProduct: function () {
			this.loaded = false;

			Api.get( `${ baseUrl }/products/${ this.$route.params.slug }`, {
				params: { demo: currentDemo }
			} )
				.then( response => {
					this.product = response.data.product;
					this.relatedProducts = response.data.relatedProducts;
					this.featuredProducts = response.data.featuredProducts;
					this.bestProducts = response.data.bestSellingProducts;
					this.latestProducts = response.data.latestProducts;
					this.topRatedProducts = response.data.topRatedProducts;
					this.prevProduct = response.data.prevProduct;
					this.nextProduct = response.data.nextProduct;

					this.product.product_categories.map( item => {
						this.productCategory.push( item );
					} );

					this.loaded = true;
				} )
				.catch( error => ( { error: JSON.stringify( error ) } ) );
		},
		hideSidebar: function () {
			document.querySelector( 'body' ).classList.remove( 'sidebar-opened' );
		},
		showSidebar: function () {
			document.querySelector( 'body' ).classList.add( 'sidebar-opened' );
		}
	}
};
</script>