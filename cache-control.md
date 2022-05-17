# Cache control

### Nginx

Nginx doesn't cache response when:

-   `private`, `no-cache` or `no-store` directives are used.
-   `set-cookie` headers is exists in response.
-   `proxy_buffering` is set to `off`.
-   `max-age` is not specified or `max-age=0` or `s-maxage=0`.

Nginx caches a response only if the origin server includes either the `expires` header with a date and time in the future, or the `cache-control` header with the `max-age` directive set to a non‑zero value.

Nginx does not use conditional headers, like `if-modified-since`, etc., in the client request.

Nginx doesn't process `cache-control` header from client request, so it is impossible to use `no-cache` to skip nginx vache.

If `must-revalidate`, `proxy-revalidate` are used nginx sends request to backend without `if-modified-since` header. If content is not modified - returns `304` to the browser.

### Browser

If `must-revalidate`, `proxy-revalidate` are used - browser sends `cache-control: max-age=0` in request.

#### Chrome

-   On `F5` chrome always revalidates main html page.
-   When chrome revalidates content it sends `cache-control: max-age=0` and `if-modified-since`.

### Configuration examples

#### Always revalidate

Cache for 1 second, revalidate:

```text
cache-control: public, max-age=1
```

#### Cache forever

```text
cache-control: public, max-age=30672000, immutable
```

#### Do not cache

```text
cache-control: no-cache, no-store
```
