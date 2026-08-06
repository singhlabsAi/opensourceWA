# Product Catalog / Menu on WhatsApp (Future Feature)

## What
Let a business (e.g. a restaurant) show customers a browsable menu/catalog directly inside WhatsApp. Customer taps something like "View Menu" → receives a list of products with photos, descriptions, and prices → taps to see more detail → can select item(s) and send them back as an order, all within the chat.

## Why
Currently the codebase only supports `sendInteractiveButtons` and `sendInteractiveList` (`src/lib/whatsapp/meta-api.ts`) — plain text/button/list interactive messages with no product data, images, or pricing attached. There's no catalog/product message type, no Meta Commerce Catalog linkage, and no handling for incoming `order` webhook events. A restaurant wanting customers to browse a real menu with images and prices can't do that today.

## How it works on WhatsApp's side
1. **Meta Commerce Catalog**: menu items (name, photo, price, description) are uploaded to a Facebook Commerce Manager catalog, linked to the business's WABA. Each item gets a `retailer_id`.
2. **Message types** (WhatsApp Cloud API, `interactive` message):
   - `product` — single item
   - `product_list` — multiple items grouped into sections (this is the "menu")
   - `catalog_message` — lets the customer browse the entire catalog inline
3. **Flow**: customer sends/taps a trigger (e.g. a button "🍽️ View Menu") → webhook fires → automation replies with a `product_list` interactive message referencing the catalog's `retailer_id`s → customer taps items, sees photo/price/description, adds to cart in WhatsApp UI → sends cart → business receives an `order` webhook event with selected items.

## What needs to be built here
1. **Catalog linkage**: store a Meta Commerce Catalog ID against the account's WhatsApp config (extend `src/app/api/whatsapp/config/route.ts` + underlying table).
2. **New API helper(s)** in `meta-api.ts`, e.g. `sendProductList()` / `sendCatalogMessage()`, following the existing `sendInteractiveList` pattern but with `interactive.type: "product_list"` and `action.catalog_id` / `action.sections[].product_items[].product_retailer_id`.
3. **Flow builder node**: a new "Send Catalog / Menu" step type in `flow-canvas.tsx` so automations can trigger it off a button/list reply.
4. **Order webhook handling**: `src/app/api/whatsapp/webhook/route.ts` needs to handle incoming `order` message type so selected items can be captured/logged/processed (e.g. forwarded to inbox, saved as an order record).

## In-CRM catalog creation & management (new tab)
Instead of requiring the business owner to set up the catalog manually in Facebook Commerce Manager, the CRM can create and manage the catalog itself via the Graph API:
1. **New "Catalog" tab** in the dashboard (alongside existing tabs like automations/settings) where the owner uploads their menu — item name, photo, price, description — directly in the CRM.
2. **Catalog creation**: on first use, backend calls `POST /{business_id}/owned_product_catalogs` to create the catalog, and saves its ID against the account's WhatsApp config (same table as the catalog linkage in the section above).
3. **Product CRUD**: form submissions call `POST /{catalog_id}/products` (and update/delete equivalents) to push each menu item to Meta, storing the returned `retailer_id` against the local item record.
4. **Image handling**: Meta does not host images itself — the product payload just needs a public HTTPS `image_url` that Meta's crawler fetches and caches on its side. Meta doesn't care where that URL is served from, so our existing **Cloudflare storage space can be used directly** — no need for a separate image host. Requirements for the URL:
   - Public, no-auth access (Meta's crawler can't send login/signed-cookie headers)
   - HTTPS (plain HTTP may be rejected)
   - Stable URL per item (replacing/deleting the image without re-syncing the catalog leaves Meta's cache stale)
   - Correct `Content-Type` header (image/jpeg, image/png, etc.)
   - Reasonable file size (avoid very large images)
   If using Cloudflare R2, bind a public bucket or custom domain so items resolve to a direct public URL (e.g. `https://cdn.yourapp.com/menu/item123.jpg`) to put in the product's `image_url` field.
5. **Permissions**: requires the `catalog_management` scope on the Meta app/WABA — needs to be added to the app's permission request during onboarding.

This makes the feature end-to-end: owner uploads menu in CRM → catalog is created/managed on Meta automatically → `sendProductList()` sends it to customers → orders come back via webhook.

## Scope note
Requires a Meta Commerce Catalog to actually exist and be connected per-account. With the in-CRM tab above, that setup happens through the product itself rather than external Facebook Commerce Manager steps — but it's still a substantial feature: new tab/UI, product CRUD + image hosting, Meta permission scope, catalog↔account linkage, plus the send/webhook plumbing described earlier.
