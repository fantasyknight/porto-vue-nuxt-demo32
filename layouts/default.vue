<template>
	<div @click="hideMobileSearch">
		<div class="page-wrapper">
			<pv-header></pv-header>
			<nuxt></nuxt>
			<pv-footer></pv-footer>

			<div class="wishlist-popup">
				<div class="wishlist-popup-msg">Product added!</div>
			</div>
		</div>

		<pv-cart-popup class="minipopup-area"></pv-cart-popup>

		<a
			id="scroll-top"
			href="javascript:;"
			title="Top"
			role="button"
			@click="scrollToTop"
		>
			<i class="icon-angle-up"></i>
		</a>

		<pv-mobile-menu></pv-mobile-menu>

		<pv-sticky-footer></pv-sticky-footer>
	</div>
</template>

<script>
import PvHeader from '~/components/common/PvHeader';
import PvFooter from '~/components/common/PvFooter';
import PvCartPopup from '~/components/common/partials/PvCartPopup';
import PvMobileMenu from '~/components/common/partials/PvMobileMenu';
import PvStickyFooter from '~/components/common/partials/PvStickyFooter';
import {
	scrollTopHandler,
	stickyHeaderHandler,
	resizeHandler,
	showScrollTopHandler,
	scrollTo,
	setCookie,
	getCookie,
	scrollTop
} from '~/utils';

export default {
	components: {
		PvHeader,
		PvFooter,
		PvCartPopup,
		PvMobileMenu,
		PvStickyFooter
	},
	watch: {
		$route: function () {
			resizeHandler();
			this.$modal.hideAll();
		}
	},
	mounted: function () {
		window.addEventListener( 'scroll', stickyHeaderHandler, {
			passive: true
		} );
		window.addEventListener( 'scroll', showScrollTopHandler, {
			passive: true
		} );
		window.addEventListener( 'resize', stickyHeaderHandler );
		window.addEventListener( 'resize', resizeHandler );
	},
	destroyed: function () {
		window.removeEventListener( 'scroll', showScrollTopHandler, {
			passive: true
		} );
		window.removeEventListener( 'scroll', stickyHeaderHandler, {
			passive: true
		} );
		window.removeEventListener( 'resize', stickyHeaderHandler );
		window.removeEventListener( 'resize', resizeHandler );
	},
	methods: {
		scrollToTop: function () {
			scrollTop( false, 70 );
		},
		hideMobileSearch: function () {
			if ( document.querySelector( '.header-search' ) ) {
				let headerSearch = document.querySelectorAll( '.header-search' );

				headerSearch.forEach( item => {
					item.classList.remove( 'show' );
					item.querySelector( '.header-search-wrapper' ).classList.remove( 'show' );
				} )
			}
		}
	}
};
</script>