# Cache control

### Nginx

Nginx doesn't cache response when:

-   `private`, `no-cache` or `no-store` directives are used.
-   `Set-Cookie` headers is exists in response.
-   `proxy_buffering` is set to `off`.
-   `max-age=0` or `s-maxage=0`.

If `must-revalidate`, `proxy-revalidate` are used nginx sends request to backend without `If-Modified-Since` header. If content is not modified - returns `304` to the browser.

### Browser

If `must-revalidate`, `proxy-revalidate` are used - browser sends `Cache-Control: max-age=0` in request.

### Configuration examples

#### Always revalidate

For `nginx` the only working solution is to use `X-Accel-Expires` header. `X-Accel-Expires` header must always placed before `Cache-Control`.

```text
X-Accel-Expires: @1
Cache-Control: public, max-age=0
```

#### Cache forever

```text
Cache-Control: public, max-age=30672000, immutable
```

#### Do not cache

```text
Cache-Control: no-store, no-cache
```
