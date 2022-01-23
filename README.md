# Cloudflare CORS Anywhere Proxy Worker

Cloudflare CORS proxy in a worker.

This script is heavily inspired by [cors-anywhere](https://github.com/Rob--W/cors-anywhere) by [Rob--W](https://github.com/Rob--W) and [cloudflare-cors-anywhere](https://github.com/Zibri/cloudflare-cors-anywhere) by [Zibri](https://github.com/Zibri).

It follows the `xo` code style.

For the most part the only 'special' feature is that it supports api keys in the header `x-cors-proxy-api-key`. You can provide an object of valid API Keys which will be checked against.

These API Keys can also be set expired, expireAt a certain date, and include indivudual exclude destination urls and include (allow) only certain origin urls.

```js
const apiKeys = {
	EZWTLwVEqFnaycMzdhBz: {
		name: 'Test App',
		expired: false,
		expiresAt: new Date('2023-01-01'),
		exclude: [], // Regexp for blacklisted urls
		include: ['.*'], // Regexp for whitelisted origins
	},
};
```

When you deploy this Cloudflare Worker this is how the URLs work. Request examples:

- http://{YOUR_WORKER_URL}/http://google.com/ - Google.com with CORS headers
- http://{YOUR_WORKER_URL}/google.com - Same as previous.
- http://{YOUR_WORKER_URL}/google.com:443 - Proxies https://google.com/
- http://{YOUR_WORKER_URL}/ - Shows usage text
- http://{YOUR_WORKER_URL}/favicon.ico - Replies 204 No Content
- http://{YOUR_WORKER_URL}/robots.txt - Replies 204 No Content

Calling the CORS Proxy via javascript. Example Request:

```js
fetch('https://{YOUR_WORKER_URL}/?https://httpbin.org/post', {
  method: 'post',
  headers: {
    'x-cors-proxy-api-key': '{ONE_OF_THE_API_KEYS}',
    'x-foo': 'bar',
    'x-bar': 'foo',
    'x-cors-headers': JSON.stringify({
      // allows to send forbidden headers
      // https://developer.mozilla.org/en-US/docs/Glossary/Forbidden_header_name
      'cookies': 'x=123'
    })
  }
}).then(res => {
  // allows to read all headers (even forbidden headers like set-cookies)
  const headers = JSON.parse(res.headers.get('cors-received-headers'))
  console.log(headers)
  return res.json()
}).then(console.log)
```
