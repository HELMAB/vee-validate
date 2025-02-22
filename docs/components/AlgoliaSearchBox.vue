<template>
  <div id="docsearch" class="lg:w-full mx-5" />
</template>

<script>
import docsearch from '@docsearch/js';

function isSpecialClick(event) {
  return event.button === 1 || event.altKey || event.ctrlKey || event.metaKey || event.shiftKey;
}

export default {
  data() {
    return {
      placeholder: undefined,
    };
  },
  mounted() {
    this.initialize(this.$config.algolia, 'en');
  },
  methods: {
    stripTrailingSlash(url) {
      return url.replace(/\/$|\/(?=\?)|\/(?=#)/g, '');
    },
    getRelativePath(absoluteUrl) {
      const { pathname, hash } = new URL(absoluteUrl);
      const url = pathname + hash;

      return this.stripTrailingSlash(url);
    },
    initialize(userOptions) {
      docsearch({
        ...userOptions,
        container: '#docsearch',
        debug: process.env.NODE_ENV !== 'production',
        navigator: {
          navigate: evt => {
            const { itemUrl } = evt;
            const { pathname: hitPathname } = new URL(window.location.origin + itemUrl);

            // Vue Router doesn't handle same-page navigation so we use
            // the native browser location API for anchor navigation.
            if (this.$router.history.current.path === hitPathname) {
              window.location.assign(window.location.origin + itemUrl);
              return;
            }

            this.$router.push(itemUrl.replace('/v4', '/'));
          },
        },
        transformItems: items => {
          return items.map(item => {
            return {
              ...item,
              url: this.getRelativePath(item.url),
            };
          });
        },
        hitComponent: ({ hit, children }) => {
          return {
            type: 'a',
            ref: undefined,
            constructor: undefined,
            key: undefined,
            props: {
              href: hit.url,
              onClick: event => {
                if (isSpecialClick(event)) {
                  return;
                }
                // We rely on the native link scrolling when user is
                // already on the right anchor because Vue Router doesn't
                // support duplicated history entries.
                if (this.$router.history.current.fullPath === hit.url) {
                  return;
                }

                const { pathname, hash } = new URL(window.location.origin + hit.url);
                // If the hits goes to another page, we prevent the native link behavior
                // to leverage the Vue Router loading feature.
                if (this.$router.history.current.path !== pathname) {
                  event.preventDefault();
                }
                // makes sure to remove base path to prevent duplicate base paths
                const url = pathname.replace('/v4/', '/') + hash;
                this.$router.push(url);
              },
              children,
            },
            __v: null,
          };
        },
      });
    },
    update(options, lang) {
      this.$el.innerHTML = '<input id="algolia-search-input" class="search-query">';
      this.initialize(options, lang);
    },
  },
  watch: {
    $lang(newValue) {
      this.update(this.options, newValue);
    },
    options(newValue) {
      this.update(newValue, this.$lang);
    },
  },
};
</script>

<style lang="postcss">
@import '@docsearch/css';

.DocSearch {
  font-family: Arial, Helvetica, sans-serif;
  --docsearch-primary-color: var(--accent);
  --docsearch-highlight-color: var(--docsearch-primary-color);
  --docsearch-text-color: var(--color-gray-200);
  --docsearch-modal-background: #f6f6f6;
  --docsearch-searchbox-shadow: inset 0 0 0 2px var(--docsearch-primary-color);
  --docsearch-searchbox-background: #e8e8e8;
  --docsearch-searchbox-focus-background: #e8e8e8;
  --docsearch-hit-color: var(--color-gray-200);
  --docsearch-muted-color: var(--color-gray-500);
  --docsearch-logo-color: var(--accent);
}

.DocSearch-Button {
  @apply w-full ml-0 rounded-md px-3 !important;
}

.DocSearch-Button-Placeholder {
  @apply px-3 !important;
}

.DocSearch-Screen-Icon > svg {
  display: inline !important;
}

.dark {
  .DocSearch {
    --docsearch-text-color: #fff;
    --docsearch-container-background: rgba(9, 10, 17, 0.8);
    --docsearch-modal-background: hsl(240 6% 9%);
    --docsearch-modal-shadow: inset 1px 1px 0 0 #2c2e40, 0 3px 8px 0 #000309;
    --docsearch-searchbox-background: hsl(0 0% 29%);
    --docsearch-searchbox-focus-background: hsl(0 0% 29%);
    --docsearch-hit-color: #fff;
    --docsearch-hit-shadow: none;
    --docsearch-hit-background: #333;
    --docsearch-key-gradient: linear-gradient(-26.5deg, #161618, #4a4a4a);
    --docsearch-key-shadow: inset 0 -2px 0 0 #5a6069, inset 0 0 1px 1px #959595, 0 2px 2px 0 rgba(3, 4, 9, 0.3);
    --docsearch-footer-background: hsl(240 6% 9%);
    --docsearch-footer-shadow: inset 0 1px 0 0 rgba(73, 76, 106, 0.5), 0 -4px 8px 0 rgba(0, 0, 0, 0.2);
    --docsearch-logo-color: var(--accent);
    --docsearch-muted-color: var(--color-gray-500);
  }

  .DocSearch-NoResults,
  .DocSearch-Footer,
  .DocSearch-Help,
  .DocSearch-Reset {
    color: #e8e8e8;
  }
}
</style>
