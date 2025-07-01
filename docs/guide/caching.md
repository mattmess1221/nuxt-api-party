# Caching

If your endpoint supports [client caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Caching), the response will be cached by the browser. Subsequent GET requests to the same endpoint will return the cached response, if available, without making a new request to the server.

The cached response will be used until it expires.

::: info
Your upstream server endpoints **MUST** respond with a `Cache-Control` header for caching to be enabled.
:::

## Cache Options

Caching is enabled by default for all requests. You can control the caching behavior by setting the `cache` option in the request options. The `cache` option accepts the same values as [`Request.cache`](https://developer.mozilla.org/en-US/docs/Web/API/Request/cache).

The available options are:

- `'default'`: Checks the cache. If it is missing or stale, executes the request and stores the cached response.
- `'no-store'`: Always fetch from the server, doesn't update the cache.
- `'reload'`: Reload the resource from the server and update the cache.
- `'no-cache'`: Use the cache, but revalidate with the server before returning the cached response.
- `'force-cache'`: Use the cache, even if it is stale.
- `'only-if-cached'`: Use the cache, but do not make a request to the server if the resource is not in the cache. If the resource is not in the cache, will respond with a 504 Gateway Timeout error.
- `true`: Equivalent to `'default'`
- `false`: Equivalent to `'no-store'`

Option comparison matrix:

| Cache Option      | loads cache | stores cache | reuses stale | makes request
| ------------------|-------------|--------------|--------------|--------------
| `'default'`       | ✅          | ✅          | ❌           | ✅
| `'no-store'`      | ❌          | ❌          | ❌           | ✅
| `'reload'`        | ❌          | ✅          | ❌           | ✅
| `'no-cache'`      | ✅          | ❌          | ❌           | ✅
| `'force-cache'`   | ✅          | ❌          | ✅           | ✅
| `'only-if-cached'`| ✅          | ❌          | ✅           | ❌

## Examples

::: info
These examples assume that you have set up an API endpoint called `jsonPlaceholder`:

```ts
// `nuxt.config.ts`
export default defineNuxtConfig({
  modules: ['nuxt-api-party'],

  apiParty: {
    endpoints: {
      jsonPlaceholder: {
        url: 'https://jsonplaceholder.typicode.com'
      }
    }
  }
})
```

:::

### Disable caching

```ts
// Disable caching for a single request
const { data } = await useJsonPlaceholderData('posts/1', {
  cache: 'no-store' // [!code ++]
})
```

### Refresh cached data

For resources that may change frequently, use the `'no-cache'` option to ensure that the browser checks with the server for a fresh response before returning the cached response.

```ts
const { data, refresh } = await useJsonPlaceholderData('posts', {
  cache: 'no-cache' // [!code ++]
})

async function invalidateAndRefresh() {
  await refresh()
}
```
