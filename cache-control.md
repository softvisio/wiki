# Cache control

### Nginx

Nginx doesn't cache response when:

-   `private`, `no-cache` or `no-store` directives are used.
-   `set-cookie` headers is exists in response.
-   `proxy_buffering` is set to `off`.
-   `max-age=0` or `s-maxage=0`.

If `must-revalidate`, `proxy-revalidate` are used nginx sends request to backend without `if-modified-since` header. If content is not modified - returns `304` to the browser.

### Browser

If `must-revalidate`, `proxy-revalidate` are used - browser sends `cache-control: max-age=0` in request.

#### Chrome

-   On `F5` chrome always revalidates main html page.
-   When chrome revalidates content it sends `cache-control: max-age=0` and `if-modified-since`.

### Configuration examples

#### Always revalidate

For `nginx` the only working solution is to use `x-accel-expires` header. `X-Accel-Expires` header must always placed before `Cache-Control`.

```text
x-accel-expires: @1
cache-control: public, max-age=0
```

#### Cache forever

```text
cache-control: public, max-age=30672000, immutable
```

#### Do not cache

```text
cache-control: no-cache, no-store
```
