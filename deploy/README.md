# Euphorie website

The public homepage source is `frontend/index.html`.
The website source checkout is separate from the older application checkout because their Git histories differ.

## Deployment

The existing EC2 host serves `/home/ubuntu/Euphorie/frontend/index.html` with Nginx.
Publish only the reviewed website files into that existing application directory.
Keep the current file permissions and assign a fresh modification time when replacing HTML.
Preserving an old modification time can cause incorrect `304 Not Modified` responses.

The main HTTPS server block includes:

```nginx
include /etc/nginx/snippets/euphorie-homepage-cache.conf;
```

Install the reviewed `deploy/nginx/homepage-cache.conf` as the root-owned
`/etc/nginx/snippets/euphorie-homepage-cache.conf`.
The include applies revalidation to `/` and `/index.html`, including requests with query parameters.
It uses negative expiry to send `Cache-Control: no-cache` and a past `Expires` value.
Existing image, manifest, API, and TLS configuration stays in the host's vhost.
Validate changes with `nginx -t`, then reload Nginx gracefully.
Keep vhost backups outside `sites-enabled` so Nginx does not load them as extra configuration.

## Validation

Confirm that both homepage paths serve the same committed HTML and `Cache-Control: no-cache`.
A request with an April 2026 `If-Modified-Since` value must receive the new HTML with status `200`.
A valid current conditional request can receive `304` normally.

The older application checkout can contain local backend and Flutter work.
Do not replace its history or stage those files as part of a website update.
