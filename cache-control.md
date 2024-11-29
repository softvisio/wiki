# Cache control

### Nginx

Nginx doesn't cache response when:

- `private`, `no-cache` or `no-store` directives are used.
- `set-cookie` headers is exists in response.
- `proxy_buffering` is set to `off`.
- `max-age` is not specified or `max-age=0` or `s-maxage=0`.

Nginx caches a response only if the origin server includes either the `expires` header with a date and time in the future, or the `cache-control` header with the `max-age` directive set to a non‑zero value.

Nginx compares client request `if-modified-since` header with the cached `last-modified` header value, returns `304` if content wasn't updated.

Nginx doesn't process `cache-control` header from client request, so it is impossible to use `no-cache` to skip nginx cache. The only way is to use `proxy_cache_bypass` directive.

### Chrome

- On `F5` chrome always revalidates main html page.

- When chrome revalidates content it sends `cache-control: max-age=0` and `if-modified-since`.

- If `must-revalidate`, `proxy-revalidate` are used - browser sends `cache-control: max-age=0` in request.

### Configuration examples

#### Cache for 1 second, revalidate

```text
cache-control: public, max-age=1
```

#### Cache forever

```text
cache-control: public, max-age=30672000
```

#### Do not cache

```text
cache-control: no-cache, no-store
```
