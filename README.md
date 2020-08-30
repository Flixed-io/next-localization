<h1 align="center">
	next-localization
	<a href="https://www.npmjs.org/package/next-localization"><img src="https://img.shields.io/npm/v/next-localization.svg?style=flat" alt="npm"></a>
    <a target="_blank" rel="noopener noreferrer" href="https://github.com/StarpTech/next-localization/workflows/Continuous%20Integration/badge.svg"><img src="https://github.com/StarpTech/next-localization/workflows/Continuous%20Integration/badge.svg" alt="Continuous Integration" style="max-width:100%;"></a>
</h1>
<p align="center">The <strong>minimalistic</strong> localization solution for <em><a href="https://github.com/vercel/next.js">Next.js</a></em>, powered by <a href="https://github.com/lukeed/rosetta">Rosetta</a></p>

---

## ✨ Features <a name="features"></a>

-   Supports all rendering modes: (Static) | ● (SSG) | λ (Server).
-   Less than 500 bytes – including dependencies!
-   No build process, No conventions.
-   Works with React hooks.

## Installation & Setup <a name="setup"></a> <a name="installation"></a>

```
yarn add next-localization
```

See [`Demo`](./example) for full example and locale setup.

## Basic Usage

Your `_app.js`.

```js
import { I18nProvider } from 'next-localization';

export default function MyApp({ Component, pageProps }) {
    return (
        <I18nProvider lngDict={{ hello: 'world', welcome: 'Welcome, {{username}}!' }} locale={'en'}>
            <Component {...pageProps} />
        </I18nProvider>
    );
}
```

Any functional component.

```js
import { useI18n } from 'next-localization';

const HomePage = () => {
    const i18n = useI18n();

    // Change locale
    i18n.locale('de'); // if dict was already loaded
    i18n.locale('de', DE); // set and load dict at the same time

    // Get current locale
    i18n.locale(); // de

    // Checkout https://github.com/lukeed/rosetta for full interpolation support
    return <p>{i18n.t('welcome', { username })}</p>;
};
```

## Usage with [`getStaticProps`](https://nextjs.org/docs/basic-features/data-fetching#getstaticprops-static-generation)

You can use Next.js's static APIs to feed your `_app.js`'s `lngDict`:

```js
import { I18nProvider } from 'next-localization';

export default function MyApp({ Component, pageProps }) {
    return (
        <I18nProvider lngDict={pageProps.lngDict} locale={pageProps.lng}>
            <Component {...pageProps} />
        </I18nProvider>
    );
}
```

Each page should define [`getStaticProps`](https://nextjs.org/docs/basic-features/data-fetching#getstaticprops-static-generation) and [`getStaticPaths`](https://nextjs.org/docs/basic-features/data-fetching#getstaticpaths-static-generation):

```js
// pages/[lng]/index.js

export const getStaticProps = async ({ params }) => {
    // You could also fetch from external API at build time
    // this example loads locales from disk once for each language defined in `params.lng`
    const { default: lngDict = {} } = await import(`../../locales/${params.lng}.json`);

    return {
        props: {
            lng: params?.lng,
            lngDict
        },
        // Next.js will attempt to re-generate the page:
        // - When a request comes in
        // - At most once every second
        revalidate: 1
    };
};

export const getStaticPaths = async () => {
    // You could also fetch from external API at build time
    const languages = ['en', 'de', 'fr'];

    return {
        paths: languages.map((lng) => ({ params: { lng } })),
        fallback: false
    };
};
```

_The same steps works with [`getServerSideProps`](https://nextjs.org/docs/basic-features/data-fetching#getserversideprops-server-side-rendering)._

## Redirect to default language

Next.js +9.5 is shipped with a rewrite engine. This will create a permanent redirect from `/` to `/en` when `en` should be your default language. This features requires to run Next.js in server mode. This won't work if you just export your site.

```js
module.exports = {
    redirects() {
        return [
            {
                source: '/',
                destination: '/en',
                permanent: true
            }
        ];
    }
};
```

## Pluralization

This library isn't shipped yet with pluralization rules but there exist a good [workaround](https://github.com/lukeed/rosetta/issues/4).

## Performance considerations

Don't forget that a locale change will rerender all components under the `I18nProvider` provider.
It's safe to create multiple providers with different language dictionaries. This can be useful for lazy-loading scenarios. For all other cases, you can still use `React.memo`, `useMemo` in your components.
