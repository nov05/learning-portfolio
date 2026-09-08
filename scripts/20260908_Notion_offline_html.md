# Save Notion page to an offline HTML file

* In the browser, press F12, run the following script in the console.  
* Agenda 那个下箭头对应的内容，一定要在运行脚本前先展开。  

```javascript
(() => {
  /*
   * Notion → Standalone Offline HTML
   * Dark mode + links + toggles + compact tables
   */

  // ---------- Helpers ----------

  const escapeHtml = (str) =>
    String(str ?? "")
      .replace(/&/g, "&amp;")
      .replace(/</g, "&lt;")
      .replace(/>/g, "&gt;")
      .replace(/"/g, "&quot;")
      .replace(/'/g, "&#39;");

  const absoluteUrl = (url) => {
    try {
      return new URL(url, location.href).href;
    } catch {
      return url;
    }
  };

  // ---------- Clone the visible Notion page ----------

  const app = document.querySelector("#notion-app");

  if (!app) {
    alert("没有找到 Notion 页面。请确认你是在原始 Notion 页面里运行此脚本。");
    return;
  }

  /*
   * Notion has a lot of UI outside the actual page.
   * We try to identify the main page content.
   */
  let source =
    document.querySelector('[class*="notion-page-content"]') ||
    document.querySelector('[class*="notion-page"]') ||
    document.querySelector('main') ||
    app;

  // Clone so we don't modify the live Notion page.
  const clone = source.cloneNode(true);

  // ---------- Remove unnecessary UI ----------

  const removeSelectors = [
    'script',
    'style',
    'noscript',
    'iframe',
    'button',
    '[contenteditable="true"]',
    '[aria-label*="comment" i]',
    '[aria-label*="share" i]',
    '[aria-label*="favorite" i]',
    '[aria-label*="add to" i]',
    '[data-testid*="toolbar" i]',
    '[data-testid*="sidebar" i]',
    '[class*="sidebar" i]',
    '[class*="topbar" i]',
    '[class*="toolbar" i]',
    '[class*="breadcrumb" i]'
  ];

  removeSelectors.forEach(sel => {
    clone.querySelectorAll(sel).forEach(el => el.remove());
  });

  // ---------- Fix links ----------

  clone.querySelectorAll("a").forEach(a => {
    const href = a.getAttribute("href");

    if (href) {
      a.setAttribute("href", absoluteUrl(href));
      a.setAttribute("target", "_blank");
      a.setAttribute("rel", "noopener noreferrer");
    }
  });

  // ---------- Convert images/icons safely ----------

  /*
   * Notion frequently uses SVG/icon URLs such as:
   * /icons/subtitles_gray.svg?mode=dark
   *
   * Those URLs break offline.
   *
   * For the small toggle/arrow icons we replace them with
   * an inline SVG.
   */

  const arrowSvg = `
    <svg
      class="offline-toggle-arrow"
      width="16"
      height="16"
      viewBox="0 0 16 16"
      xmlns="http://www.w3.org/2000/svg"
      aria-hidden="true"
    >
      <path
        d="M5.5 3.5L10 8L5.5 12.5"
        fill="none"
        stroke="currentColor"
        stroke-width="1.6"
        stroke-linecap="round"
        stroke-linejoin="round"
      />
    </svg>
  `;

  /*
   * Replace broken/local Notion SVG references.
   */
  clone.querySelectorAll("img").forEach(img => {
    const src = img.getAttribute("src") || "";
    const alt = (img.getAttribute("alt") || "").toLowerCase();

    if (
      src.includes("/icons/") ||
      src.includes("toggle") ||
      src.includes("chevron") ||
      src.includes("arrow") ||
      alt.includes("toggle") ||
      alt.includes("arrow")
    ) {
      const span = document.createElement("span");
      span.className = "offline-inline-icon";
      span.innerHTML = arrowSvg;
      img.replaceWith(span);
    } else if (src) {
      /*
       * Normal images are preserved as absolute URLs.
       * Your current page apparently has no important images,
       * but this prevents normal images from becoming relative URLs.
       */
      img.setAttribute("src", absoluteUrl(src));
    }
  });

  // ---------- Preserve toggle / details structures ----------

  /*
   * If the browser DOM contains <details>, keep them.
   */
  clone.querySelectorAll("details").forEach(details => {
    details.setAttribute("open", "");
  });

  /*
   * Notion toggles often use role="button".
   * We preserve their visible content rather than relying on
   * Notion's JavaScript.
   */
  clone.querySelectorAll('[role="button"]').forEach(el => {
    const text = (el.innerText || "").trim();

    if (
      text &&
      (
        el.querySelector("svg") ||
        el.querySelector("img") ||
        el.getAttribute("aria-expanded") !== null
      )
    ) {
      el.classList.add("offline-toggle-header");

      const expanded =
        el.getAttribute("aria-expanded") === "true";

      el.setAttribute("aria-expanded", expanded ? "true" : "false");
    }
  });

  // ---------- Convert tables ----------

  clone.querySelectorAll("table").forEach(table => {
    table.classList.add("offline-table");

    /*
     * Put every table inside a scroll container.
     */
    if (!table.parentElement.classList.contains("offline-table-wrap")) {
      const wrapper = document.createElement("div");
      wrapper.className = "offline-table-wrap";

      table.parentNode.insertBefore(wrapper, table);
      wrapper.appendChild(table);
    }
  });

  /*
   * Notion sometimes creates table-like div structures rather than
   * ordinary <table>. We avoid aggressively converting those because
   * doing so can destroy formatting.
   */

  // ---------- Clean inline styles that fight dark mode ----------

  clone.querySelectorAll("*").forEach(el => {
    const style = el.getAttribute("style");

    if (!style) return;

    let cleaned = style
      .replace(/background-color\s*:[^;]+;?/gi, "")
      .replace(/background\s*:[^;]+;?/gi, "")
      .replace(/color\s*:[^;]+;?/gi, "")
      .replace(/box-shadow\s*:[^;]+;?/gi, "");

    if (cleaned.trim()) {
      el.setAttribute("style", cleaned);
    } else {
      el.removeAttribute("style");
    }
  });

  // ---------- Remove Notion-specific empty containers ----------

  clone.querySelectorAll("div").forEach(div => {
    const text = (div.innerText || "").trim();

    /*
     * Don't remove anything containing meaningful children.
     */
    if (
      !text &&
      div.children.length === 0 &&
      !div.querySelector("img,svg,canvas")
    ) {
      div.remove();
    }
  });

  // ---------- Build standalone document ----------

  const title =
    document.title ||
    "Mangusta Capital Externships";

  const html = `<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>${escapeHtml(title)}</title>

<style>

/* =========================================================
   Base
   ========================================================= */

html {
  background: #191919;
}

body {
  margin: 0;
  padding: 0;
  background: #191919;
  color: #e6e6e6;

  font-family:
    -apple-system,
    BlinkMacSystemFont,
    "Segoe UI",
    Helvetica,
    Arial,
    sans-serif;

  font-size: 15px;
  line-height: 1.6;
}

/* =========================================================
   Main page
   ========================================================= */

#offline-page {
  box-sizing: border-box;

  width: 100%;
  max-width: 1200px;

  margin: 0 auto;
  padding: 32px 24px 80px;
}

/* =========================================================
   Text
   ========================================================= */

p,
div,
span,
li {
  color: inherit;
}

h1,
h2,
h3,
h4,
h5,
h6 {
  color: #f0f0f0;
  line-height: 1.3;
}

h1 {
  font-size: 32px;
  margin-top: 0;
}

h2 {
  font-size: 26px;
}

h3 {
  font-size: 21px;
}

h4 {
  font-size: 18px;
}

/* =========================================================
   Links
   ========================================================= */

a {
  color: #5da9ff;
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}

/* =========================================================
   Lists
   ========================================================= */

ul,
ol {
  padding-left: 28px;
}

/* =========================================================
   Code
   ========================================================= */

code {
  background: #2b2b2b;
  color: #f0f0f0;

  border-radius: 4px;
  padding: 2px 5px;

  font-family:
    ui-monospace,
    SFMono-Regular,
    Menlo,
    Monaco,
    Consolas,
    monospace;
}

/* =========================================================
   Blockquote
   ========================================================= */

blockquote {
  margin: 12px 0;
  padding: 8px 16px;

  border-left: 3px solid #555;

  color: #bdbdbd;
}

/* =========================================================
   Images
   ========================================================= */

img {
  max-width: 100%;
  height: auto;
}

/* =========================================================
   Notion blocks
   ========================================================= */

.notion-selectable-halo,
.notion-focusable {
  outline: none !important;
}

/* =========================================================
   Toggle / Agenda
   ========================================================= */

.offline-toggle-header {
  display: flex;
  align-items: center;
  gap: 6px;

  cursor: pointer;

  min-height: 28px;
  margin: 3px 0;

  color: #eeeeee;
  font-weight: 500;
}

.offline-inline-icon {
  display: inline-flex;

  width: 16px;
  height: 16px;

  flex: 0 0 16px;

  align-items: center;
  justify-content: center;

  color: #bdbdbd;
}

.offline-toggle-arrow {
  display: block;
}

/*
 * Preserve visible toggle contents.
 */
details {
  margin: 4px 0;
}

summary {
  cursor: pointer;
  color: #eeeeee;
}

details[open] > summary {
  margin-bottom: 4px;
}

/* =========================================================
   Tables
   ========================================================= */

/*
 * IMPORTANT:
 *
 * The table is constrained to the page width.
 * It no longer uses width:max-content,
 * so it won't balloon across the screen.
 */

.offline-table-wrap {
  width: 100%;
  max-width: 100%;

  overflow-x: auto;
  overflow-y: visible;

  margin: 8px 0 12px;

  -webkit-overflow-scrolling: touch;
}

.offline-table {
  width: 100%;
  max-width: 100%;

  table-layout: auto;

  border-collapse: collapse;
  border-spacing: 0;

  background: #1f1f1f;

  font-size: 14px;
}

/*
 * Compact table cells.
 */
.offline-table th,
.offline-table td {
  padding: 6px 8px;

  border: 1px solid #3a3a3a;

  vertical-align: top;

  /*
   * Prevent extremely long URLs / words from
   * stretching the entire table.
   */
  max-width: 320px;

  word-break: break-word;
  overflow-wrap: anywhere;

  white-space: normal;
}

/*
 * Header.
 */
.offline-table th {
  background: #292929;
  color: #f0f0f0;

  font-weight: 600;
}

/*
 * Body.
 */
.offline-table td {
  background: #1f1f1f;
  color: #dedede;
}

/*
 * Links inside tables.
 */
.offline-table a {
  color: #5da9ff;
}

/*
 * Don't let nested divs force enormous widths.
 */
.offline-table td > div,
.offline-table th > div {
  max-width: 100%;
  min-width: 0;
}

/* =========================================================
   Horizontal rules
   ========================================================= */

hr {
  border: 0;
  border-top: 1px solid #3a3a3a;
  margin: 24px 0;
}

/* =========================================================
   Selection
   ========================================================= */

::selection {
  background: #444;
  color: white;
}

/* =========================================================
   Mobile
   ========================================================= */

@media (max-width: 700px) {

  #offline-page {
    padding:
      20px
      14px
      60px;
  }

  body {
    font-size: 14px;
  }

  .offline-table {
    font-size: 13px;
  }

  .offline-table th,
  .offline-table td {
    padding: 5px 6px;
  }
}

</style>
</head>

<body>

<div id="offline-page">
${clone.outerHTML}
</div>

<script>

/*
 * Offline toggle behavior
 *
 * This makes Notion-style toggle headers clickable
 * even without Notion JavaScript.
 */

document.addEventListener("click", function(event) {

  const header =
    event.target.closest(".offline-toggle-header");

  if (!header) return;

  /*
   * Find the next meaningful sibling.
   */
  let next = header.nextElementSibling;

  if (!next) return;

  const currentlyHidden =
    next.style.display === "none";

  next.style.display =
    currentlyHidden ? "" : "none";

  header.setAttribute(
    "aria-expanded",
    currentlyHidden ? "true" : "false"
  );

  /*
   * Rotate arrow if present.
   */
  const arrow =
    header.querySelector(".offline-toggle-arrow");

  if (arrow) {
    arrow.style.transform =
      currentlyHidden
        ? "rotate(90deg)"
        : "rotate(0deg)";
  }

});

</script>

</body>
</html>`;

  // ---------- Download ----------

  const blob = new Blob(
    [html],
    { type: "text/html;charset=utf-8" }
  );

  const url = URL.createObjectURL(blob);

  const a = document.createElement("a");

  a.href = url;

  a.download =
    "Mangusta Capital Externships-offline-dark.html";

  document.body.appendChild(a);

  a.click();

  a.remove();

  setTimeout(() => {
    URL.revokeObjectURL(url);
  }, 10000);

  console.log(
    "✅ Offline HTML 已生成并开始下载。"
  );

})();
```