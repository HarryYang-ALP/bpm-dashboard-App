// 這個 service worker 只負責讓「外殼」本身（index.html / manifest / icon）
// 可以被安裝、離線時也能顯示基本畫面。
// 裡面 iframe 嵌入的 Streamlit dashboard 內容完全不快取，
// 確保每次打開都是抓最新的即時資料，不會看到過期的快照。

const CACHE_NAME = "bpm-app-shell-v1";
const SHELL_FILES = [
  "./",
  "./index.html",
  "./manifest.json",
  "./icons/icon-192.png",
  "./icons/icon-512.png",
];

self.addEventListener("install", (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => cache.addAll(SHELL_FILES))
  );
  self.skipWaiting();
});

self.addEventListener("activate", (event) => {
  event.waitUntil(
    caches.keys().then((keys) =>
      Promise.all(
        keys
          .filter((key) => key !== CACHE_NAME)
          .map((key) => caches.delete(key))
      )
    )
  );
  self.clients.claim();
});

self.addEventListener("fetch", (event) => {
  const url = new URL(event.request.url);

  // 只攔截「同源」的外殼檔案請求，走 cache-first。
  // 跨網域的請求（例如 Streamlit / Notion / Anthropic API）完全不經過這裡處理，
  // 直接放行給瀏覽器原生網路請求，避免快取到過期的 dashboard 資料。
  if (url.origin !== self.location.origin) {
    return;
  }

  event.respondWith(
    caches.match(event.request).then((cached) => {
      return (
        cached ||
        fetch(event.request).catch(() => {
          // 離線且沒快取到的情況，最基本的 fallback 回首頁殼
          return caches.match("./index.html");
        })
      );
    })
  );
});
