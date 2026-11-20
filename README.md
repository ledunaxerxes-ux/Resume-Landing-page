Build and publish a custom landing page for my Gumroad product nhrjdf.

Design a unique, conversion-focused page tailored to this product — fully responsive, accessible, and supporting light and dark mode. Save it as one self-contained file, landing.html. The page is sanitized and runs sandboxed: inline CSS/JS (animations, scroll effects, modals) and a Tailwind CDN work. For images and media, use only your product's own assets (run gumroad products view nhrjdf for its cover and thumbnail URLs) or CSS — external image/media hosts are blocked, and the page can't fetch external URLs or read the buyer's account. A page is capped at 500,000 characters, so inline an image as a data: URI only when it is small: a full-size photo inlined as base64 runs to megabytes and the publish is rejected.

IMPORTANT: a custom landing page REPLACES the entire product page, including the native price and "I want this!" button. Your HTML must include at least one buy element (see below) or the product becomes unpurchasable. Always end by loading the live page and clicking through to checkout to confirm the buy flow works.

Mark live values and buy buttons with data attributes that Gumroad fills in server-side:
- data-gumroad-field="name|price|description|rating|review-count" — interpolated with the product's current values. rating and review-count are left as you wrote them when reviews are hidden or the product has none, so put a fallback like "Be the first to review" inside those elements.
- data-gumroad-field="price" plus data-gumroad-option="<variant name>" — that variant/version/tier's current price instead of the product's, so never hand-type a tier price. An unknown name keeps your own text.
- data-gumroad-action="buy" — wired up to launch the Gumroad checkout. Use on any element (<a>, <button>, <div>).

For products with selection state, set the choice directly on the buy element so the checkout opens pre-selected (an invalid value silently falls back to the product defaults — it won't break the page):
- data-gumroad-option="<variant name>" — for products with variants/versions/tiers.
- data-gumroad-quantity="<integer>" — for products with quantity enabled.
- data-gumroad-price="<decimal>" — for pay-what-you-want products (major units, e.g. "9.99"). This sets ONE fixed price and sends the buyer straight to checkout.
- data-gumroad-recurrence="monthly|quarterly|biannually|yearly|every_two_years" — for membership/subscription products.

Example buy buttons:
  <a data-gumroad-action="buy">Buy now</a>
  <a data-gumroad-action="buy" data-gumroad-option="Pro" data-gumroad-recurrence="yearly">Buy Pro – $99/year</a>
  <button data-gumroad-action="buy" data-gumroad-quantity="2">Buy 2 seats</button>

For a pay-what-you-want product where the buyer should name their OWN price on the page, render a price <input> and post the chosen amount to checkout yourself (do NOT put data-gumroad-action="buy" on this button — Gumroad's delegated buy handler would intercept it before your custom price is added). The page is allowed to post a "gumroad:checkout" message to its parent with any of: variant, quantity, price, recurrence. Use variant for a variant/version/tier name. An empty price falls back to Gumroad's own price-entry step, so there is no dead end:
  <input id="gr-price" type="number" min="0" step="0.01" placeholder="9.99" />
  <button id="gr-buy" type="button">I want this</button>
  <script>
    document.getElementById("gr-buy").addEventListener("click", function () {
      var v = (document.getElementById("gr-price").value || "").trim(), params = {};
      if (v !== "") { var n = parseFloat(v); if (!isNaN(n) && n >= 0) params.price = String(n); }
      parent.postMessage({ type: "gumroad:checkout", params: params }, "*");
    });
  </script>

To collect waitlist signups before launch, add a waitlist form. It only works while the product is unpublished with its waitlist turned on, and visitors see your page before launch only if it has the form or the hook below; otherwise they get Gumroad's own waitlist page. Gumroad sends the email and writes its reply into the optional message element:
  <form data-gumroad-waitlist>
    <input type="email" name="email" required placeholder="you@example.com" />
    <button type="submit">Notify me</button>
    <p data-gumroad-waitlist-message></p>
  </form>
Or have an inline <script> on the page post { type: "gumroad:waitlist", email, requestId } (requestId a string) to its parent; Gumroad replies with a "gumroad:waitlist:result" message carrying success, message, and the same requestId.
Once the product is published, <html> gets data-gumroad-launched and the form stops working, so hide it with html[data-gumroad-launched] form[data-gumroad-waitlist] { display: none; } and keep a buy element on the page for launch day.

Links to your own Gumroad pages (your storefront, your other products) work as ordinary links — write them plainly and they open in the same tab, no target="_blank" needed. Links to any other site still need target="_blank".

Then preview, publish, and verify it with the Gumroad CLI:
- Run the real server-side sanitizer WITHOUT publishing and read what it changed: gumroad products page preview nhrjdf ./landing.html --json --no-input --non-interactive — inspect .sanitization_report. If it stripped tags or attributes your page needs (a buy element, an <input>, a <script>), fix the HTML and preview again. Do this until the report is clean so you never publish a broken page.
- Also inspect the top-level .warning from preview/publish. If it says the page has no buy element, add data-gumroad-action="buy" or a gumroad:checkout postMessage before publishing.
- Publish (or update) the page once preview is clean: gumroad products page publish nhrjdf ./landing.html --json --no-input --non-interactive — .sanitization_report reflects what actually shipped.
- Confirm it's live and find the public URL: gumroad products page url nhrjdf --json --jq '.product.landing_url' --no-input --non-interactive
- Remove the landing page and restore the default product page: gumroad products page clear nhrjdf --yes --json --no-input --non-interactive

If the gumroad CLI isn't installed: brew install antiwork/cli/gumroad (or curl -fsSL https://gumroad.com/install-cli.sh | bash), then run gumroad auth login.
