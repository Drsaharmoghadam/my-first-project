cat <<EOF > index.js
import { connect } from 'cloudflare:sockets';
const ID = 'ce3fcf50-7fcd-0321-89ce-9b509dec6864';
const P = 'cdn.xn--b6gac.eu.org';

export default {
  async fetch(r, e) {
    const u = new URL(r.url);
    if (r.headers.get('Upgrade') !== 'websocket') {
      if (u.pathname === "/" + ID) {
        const config = "vless://" + ID + "@" + u.host + ":443?encryption=none&security=tls&sni=" + u.host + "&fp=randomized&type=ws&host=" + u.host + "&path=%2F%3Fed%3D2048#Rastak-Safe-Node";
        return new Response(config, {status: 200});
      }
      return fetch(new Request('https://www.google.com'+u.pathname, r));
    }
    const [c, s] = Object.values(new WebSocketPair());
    s.accept();
    const st = new ReadableStream({start(cl){s.addEventListener('message',v=>cl.enqueue(v.data));s.addEventListener('close',()=>cl.close());s.addEventListener('error',e=>cl.error(e));}});
    st.pipeTo(new WritableStream({async write(k){
      const h = new Uint8Array([new Uint8Array(k.slice(0,1))[0], 0]);
      const t = connect({hostname: P, port: 443});
      const w = t.writable.getWriter();
      await w.write(k.slice(24)); w.releaseLock();
      t.readable.pipeTo(new WritableStream({async write(k){if(s.readyState===1)s.send(await new Blob([h,k]).arrayBuffer());}}));
    }})).catch(()=> {});
    return new Response(null, {status: 101, webSocket: c});
  }
};
EOF
