# NovaCart

An e-commerce frontend built with Next.js App Router, React, and TypeScript. NovaCart brings together product discovery, customer accounts, a server-backed cart, checkout preparation, order history, and the foundations of an administration dashboard.

The package name is `e-com`. This repository contains the frontend; a compatible external REST API is required for authentication, catalog data, uploads, and orders. Feature status below describes the checked-in frontend, not independently verified backend behavior.

## Contents

- [Features](#features)
- [Tech stack](#tech-stack)
- [Architecture](#architecture)
- [Folder structure](#folder-structure)
- [Routes](#routes)
- [State management and data fetching](#state-management-and-data-fetching)
- [API communication](#api-communication)
- [Forms and validation](#forms-and-validation)
- [Environment variables](#environment-variables)
- [Getting started](#getting-started)
- [Development workflow](#development-workflow)
- [Screenshots](#screenshots)
- [Future improvements](#future-improvements)
- [Author](#author)

## Features

### Authentication and profile

| Feature | Current implementation |
| --- | --- |
| Login and registration | Validated forms, API mutations, notifications, and navigation after success. Registration includes password confirmation and a terms agreement check. |
| Logout | Calls the logout API and clears the cached current user. |
| Session lookup | Fetches `/users/me`; a `401` response is treated as an unauthenticated session. |
| Profile | Displays account details, edits first/last name, and uploads or removes a profile photo. Uploads accept JPEG, PNG, WebP, and GIF up to 5 MB. |
| Change password | Existing-password form with new-password confirmation and an API mutation. |
| Email verification | Registration prompts users to check email, and user types contain verification fields. Verification and resend flows are not implemented in this frontend. |
| Password recovery | Forgot/reset password paths exist in constants, and login links to the forgot-password path. Corresponding pages and API methods are not implemented. |
| Protected routes | Guard components exist, but protection is only partially connected, as described below. |

`GuestGuard` wraps login/register and redirects signed-in users home. `AdminGuard` from `RoleGuard.tsx` is used on the dashboard index and redirects based on `isAdmin`. `AuthGuard` is defined but is not mounted by any route. The dashboard layout and child management routes are not wrapped in the admin guard. The current admin guard also renders authenticated users while its non-admin redirect is pending.

These are client-side navigation controls. The external API must enforce session and administrator permissions for protected data and mutations.

### Products and categories

- Landing page with featured products and category discovery.
- Product catalog with category filtering and API-driven pagination, requesting eight products per page. Category selection is reflected in `?categoryId=...`; the page number is held in component state.
- Product details with image, description, price, quantity selection, copy-link action, and add-to-cart mutation.
- Public category listing with links to filtered products.
- Product create/update forms with category selection and multipart image uploads.
- Category create/update forms with title and description fields.

Product and category deletion methods and hooks exist, but no delete UI currently invokes them. Dashboard product/category index pages are placeholders rather than management lists. Product editing uses the creation schema, so it requires an image file even though the update DTO supports partial changes.

### Cart, orders, and checkout

- Add one item from a product card or a selected quantity from product details.
- Retrieve the current cart and cart badge count from the backend.
- Display order items, line totals, and the cart total.
- Save shipping address, phone, and customer email before continuing to `/cart/payment`.
- Review shipping information and the order summary on the payment screen.
- Submit the current order through `POST /orders`, show a success notification, and return home.
- View user order history with item images, quantities, totals, and customer details.

### 💳 Payment Demonstration

NovaCart includes a payment step as part of the portfolio checkout scenario.

The current frontend does **not process real Stripe payments**. The payment page contains a demonstration video located at:

```text
public/assets/payment-ex.mp4

## 🎯 Project Scope

NovaCart is a **portfolio-focused full-stack project**, not a production e-commerce product intended for real commercial operation.

The application uses an e-commerce domain to demonstrate practical frontend and backend engineering concepts such as authentication, product and category management, cart state, orders, profile management, API integration, file uploads, validation, and deployment.

Because the project is designed primarily as a technical showcase, some workflows intentionally prioritize demonstrating implementation patterns over reproducing every business rule of a real-world online store.

For example, certain checkout, order, administration, and payment scenarios may be simplified or structured differently from what would normally be required in a production commerce system.

These decisions are part of the project's portfolio scope rather than claims that the application represents a complete commercial e-commerce solution.


### Dashboard and UI

The dashboard includes a collapsible sidebar and routes for product/category creation and editing. The overview and product/category list pages contain placeholder content. Users and Payments appear in the sidebar, but their dashboard routes are not implemented; user administration currently has repository/hooks only.

The interface uses responsive Tailwind grids, mobile navigation, reusable shadcn/ui components, Lucide icons, and toast feedback. Shared loading UI includes card, detail, form, and cart skeletons, plus a global loading page. Query retry panels, offline/session error states, empty states, and a custom not-found page are available. Error handling is not yet consistent across every view.

Static About, Contact, FAQ, Privacy, and Terms pages are included. Dark-mode styles exist, but a user-facing theme switcher is not implemented.

## Tech stack

Versions/ranges below are declared in [package.json](./package.json); `pnpm-lock.yaml` records resolved dependencies.

| Technology | Version/range | Role |
| --- | --- | --- |
| Next.js | `16.3.5` | App Router, layouts, metadata, images/fonts, API rewrites |
| React / React DOM | `19.2.8` | Rendering and local UI state |
| TypeScript | `^5` | Strict typing and domain contracts |
| Tailwind CSS / PostCSS plugin | `^4` | Styling and design tokens |
| shadcn | `^4.21.0` | Local UI components; `base-nova` configuration |
| Base UI | `^1.8.0` | Interactive UI primitives |
| React Hook Form | `^7.88.0` | Form state and field control |
| Hook Form resolvers | `^5.9.1` | Zod integration |
| Zod | `^4.6.5` | Schemas and inferred DTO types |
| TanStack React Query | `^5.103.0` | Queries, mutations, server-state cache |
| React Query Devtools | `^5.103.1` | Query inspection |
| Axios | `^1.20.0` | HTTP client |
| React Toastify | `^11.1.0` | Notifications |
| Lucide React | `^1.46.0` | Icons |
| Class Variance Authority | `^0.7.1` | Component variants |
| `cn` | `^0.3.0` | Class-name utility, re-exported through `lib/utils.ts` |
| `tw-animate-css` | `^1.4.0` | CSS animations |
| ESLint / Next.js ESLint config | `^9` / `16.3.5` | Static analysis and framework rules |
| pnpm | `10.28.2` | Package manager pinned in `packageManager` |

## Architecture

Routes are thin entry points: they provide metadata, resolve dynamic parameters, and render feature views. The root layout installs React Query and toast providers alongside the shared header/footer. Interactive views use client components.

Business functionality lives in `app/_modules`, grouped by domain. This keeps each feature's UI, API contract, validation, and data lifecycle close together.

```text
App Router page → feature view → query/mutation hook → repository → Axios
                                                                      ↓
                                                               /api rewrite
                                                                      ↓
                                                                external API
```

| Layer | Responsibility |
| --- | --- |
| `views/` | Render screens and feature components; coordinate interactions. The landing module uses singular `view/`. |
| `hooks/` | Encapsulate queries, mutations, query keys, and cache changes. |
| `repo/` | Define typed interfaces and Axios implementations, such as `products.ts` and `resproducts.ts`. |
| `dto/` | Define input schemas and inferred request/form types. |
| `entities/` | Describe domain objects and API response structures. |
| `adapters/` | Convert transport data into a view-friendly shape where needed. |
| Module `utils/` | Hold field definitions and feature-specific constants. |
| `app/_utils/` | Share Axios, API error extraction, network-error detection, and date formatting. |
| `components/` | Share UI primitives, validation inputs, guards, navigation, and skeletons. |

For example, the order adapter flattens nested user/product data and converts prices to numbers before cart and order views consume it. Repository boundaries keep endpoints and serialization out of components, while hooks centralize server-state behavior. This separation makes features easier to extend and test independently.

## Folder structure

Representative tree of the current repository; generated files and individual UI primitives are omitted.

```text
.
├── app/
│   ├── (pages)/
│   │   ├── (auth)/
│   │   │   ├── layout.tsx
│   │   │   ├── login/page.tsx
│   │   │   └── register/page.tsx
│   │   ├── about/page.tsx
│   │   ├── cart/
│   │   │   ├── page.tsx
│   │   │   └── payment/page.tsx
│   │   ├── categories/page.tsx
│   │   ├── contact/page.tsx
│   │   ├── dashboard/
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx
│   │   │   ├── categories/
│   │   │   │   ├── page.tsx
│   │   │   │   ├── add/page.tsx
│   │   │   │   └── [id]/update/page.tsx
│   │   │   └── products/
│   │   │       ├── page.tsx
│   │   │       ├── add/page.tsx
│   │   │       └── [id]/update/page.tsx
│   │   ├── faq/page.tsx
│   │   ├── orders/page.tsx
│   │   ├── privacy/page.tsx
│   │   ├── products/
│   │   │   ├── page.tsx
│   │   │   └── [id]/product-details/page.tsx
│   │   ├── profile/
│   │   │   ├── page.tsx
│   │   │   ├── update-password/page.tsx
│   │   │   └── update-profile/page.tsx
│   │   └── terms/page.tsx
│   ├── _config/env.ts
│   ├── _modules/
│   │   ├── auth/          # dto, hooks, repo, utils, views
│   │   ├── categories/    # dto, entities, hooks, repo, utils, views
│   │   ├── dashboard/     # views
│   │   ├── landing/       # view
│   │   ├── order/         # adapters, dto, entities, hooks, repo, utils, views
│   │   ├── order_item/    # dto, entities, hooks, repo
│   │   ├── products/      # dto, entities, hooks, repo, utils, views
│   │   └── user/          # dto, entities, hooks, repo, utils, views
│   ├── _providers/
│   │   ├── react-query.provider.tsx
│   │   └── toast-provider.tsx
│   ├── _types/type.ts
│   ├── _utils/
│   ├── globals.css
│   ├── layout.tsx
│   ├── loading.tsx
│   ├── not-found.tsx
│   └── page.tsx
├── components/
│   ├── guards/
│   ├── icons/
│   ├── inputs/
│   ├── sharing/
│   ├── skeletons/
│   └── ui/
├── lib/utils.ts
├── public/assets/         # Branding, illustrations, payment demo video
├── components.json
├── eslint.config.mjs
├── next.config.ts
├── package.json
├── pnpm-lock.yaml
├── pnpm-workspace.yaml
├── postcss.config.mjs
└── tsconfig.json
```

Parenthesized folders are route groups and do not appear in URLs. Underscore-prefixed folders organize private application code. The `@/*` TypeScript alias resolves from the repository root.

## Routes

| Area | Paths |
| --- | --- |
| Storefront | `/`, `/products`, `/products/[id]/product-details`, `/categories` |
| Authentication | `/login`, `/register` |
| Account | `/profile`, `/profile/update-profile`, `/profile/update-password` |
| Shopping | `/cart`, `/cart/payment`, `/orders` |
| Dashboard shell/placeholders | `/dashboard`, `/dashboard/products`, `/dashboard/categories` |
| Product forms | `/dashboard/products/add`, `/dashboard/products/[id]/update` |
| Category forms | `/dashboard/categories/add`, `/dashboard/categories/[id]/update` |
| Information | `/about`, `/contact`, `/faq`, `/privacy`, `/terms` |

## State management and data fetching

TanStack React Query manages remote data; React state manages transient UI such as selected quantity, pagination, and sidebar visibility. There is no separate Redux or Zustand store.

The shared provider configures a five-minute `staleTime`, disables window-focus refetching and automatic query retries, and sets `gcTime: 0`, removing inactive queries immediately. React Query Devtools is included in the provider.

Most domains use query-key factories:

```ts
productsQueryKeys.all                         // ["products"]
productsQueryKeys.filtered(1, 8, 2)            // ["products", "filtered", 1, 8, 2]
productsQueryKeys.one(42)                      // ["products", "single", 42]
userQueryKeys.me()                            // ["users", "me"]
orderItemQueryKeys.byOrderId(7)                // ["order_items", "by_order_id", 7]
```

Orders currently use separate keys: `["orderDetail"]`, `["orderList"]`, and `["orderCartCount"]`.

Hooks delegate network calls to repositories. Mutations use `invalidateQueries` to request fresh data and `setQueryData` for direct updates. Product/category creation invalidates the domain prefix; adding an order item invalidates order-item queries and the cart count; login invalidates the current-user query.

Cache handling still needs consolidation. Some mutation keys are nested or combined in ways that do not match their queries, and product/category updates write a single record into a root cache entry. Logout attempts to remove queries marked `meta.requiresAuth`, but existing queries do not set that metadata. These behaviors need review for consistent refresh and session-cache cleanup.

## API communication

The shared client in [app/_utils/axiosInstance.ts](./app/_utils/axiosInstance.ts) uses:

```ts
axios.create({
  baseURL: "/api",
  withCredentials: true,
});
```

[next.config.ts](./next.config.ts) forwards `/api/:path*` to `${process.env.NEXT_PUBLIC_API_URL}/:path*`. A browser request to `/api/products`, for example, reaches `<backend-base-url>/products`. These are external rewrites; the repository does not implement matching local API route handlers.

Authentication is designed around backend-managed cookies. Requests enable credentials, and the frontend retrieves identity through `/users/me`; it does not attach a bearer token or persist an authentication token in local storage. Cookie issuance, expiry, attributes, and authorization depend on the backend and deployment configuration. The backend must issue cookies compatible with the frontend/proxy origin.

| API area | Endpoints used, relative to the backend base URL |
| --- | --- |
| Authentication | `POST /users/auth/login`, `POST /users/auth/register`, `POST /users/auth/logout` |
| Profile/users | `GET /users/me`, `GET/PUT /users`, `GET/DELETE /users/:id`, `PUT /users/password/change-password` |
| Profile images | `POST /users/photo-upload`, `DELETE /users/photo-delete` |
| Products | `GET/POST /products`, `GET/PUT/DELETE /products/:id`, `GET /products/categories/:categoryId` |
| Categories | `GET/POST /categories`, `GET/PUT/DELETE /categories/:id` |
| Orders | `GET/POST /orders`, `GET/PUT /orders/:id`, `GET /orders/user-orders`, `GET /orders/cart`, `GET /orders/cart/count` |
| Order items | `GET/POST /order-items`, `GET/DELETE /order-items/:id`, `GET /order-items/orders/:orderId` |

Repository support does not imply a completed UI for every endpoint. Product lists expect `products`, `count`, and `pageCount` in the response and send `page`/`limit` query parameters. Product and profile image uploads use `FormData`. The response interceptor passes successes through and rejects errors without token refresh or retry logic.

## Forms and validation

Forms use `useForm`, `FormProvider`, and `zodResolver`. Zod schemas in each module's `dto/` directory describe inputs, and `z.infer` provides matching TypeScript types. Schemas cover authentication, product/category fields, profile changes, password changes, order details, and uploads.

Shared `ValidationInput`, `ValidationSelect`, and `ValidationCheckbox` components use `Controller` and `useFormContext` to connect UI controls to form state. They render labels, invalid-state attributes, and field errors consistently. Module `utils/fields.ts` files describe repeated field layouts. Views handle submission feedback and navigation; repositories serialize requests.

Validation is still evolving: several forms cast schemas to `any`, and product editing reuses the creation schema. Server-side validation remains the responsibility of the API.

## Environment variables

Create `.env.local` in the project root using this **`.env.example` template**:

```dotenv
# Replace with the actual backend base URL; omit the trailing slash.
# Include a backend path prefix only if your API requires one.
NEXT_PUBLIC_API_URL=http://localhost:4000
```

The address above is illustrative; this repository does not prescribe or start a backend on port 4000.

| Variable | Required | Purpose |
| --- | --- | --- |
| `NEXT_PUBLIC_API_URL` | Yes | Absolute backend base URL used by the `/api` rewrite. |

Only this application-specific environment variable is referenced. No Stripe, database, or Cloudinary secret is required by the checked-in frontend. `NEXT_PUBLIC_*` values are public configuration and must not contain secrets.

`app/_config/env.ts` contains a Zod URL validator, but it is not imported by the application/configuration, so it does not enforce startup validation. Set the variable before development or production builds and restart/rebuild after changing it. Environment files are ignored by Git; the template is kept inline here.

## Getting started

### Requirements

- Node.js **20.9.0 or newer**, the minimum required by the installed Next.js package.
- pnpm **10.28.2**, matching `package.json`.
- A running backend implementing the repository contracts. Catalog data and an administrator account must be supplied through that backend; no backend seed/setup scripts are included here.

### Installation

```bash
git clone https://github.com/momen-x/ecommerce.git
cd ecommerce
npm install --global pnpm@10.28.2
pnpm install
```

Create `.env.local` using the template above, then start the frontend:

```bash
pnpm dev
```

Open [localhost:3000](http://localhost:3000).

### Build and checks

```bash
pnpm lint
pnpm build
pnpm start
```

| Command | Purpose |
| --- | --- |
| `pnpm install` | Install dependencies. |
| `pnpm dev` | Start the Next.js development server. |
| `pnpm build` | Create a production build. |
| `pnpm start` | Serve the production build. |
| `pnpm lint` | Run ESLint. |

On Windows, if PowerShell blocks the `pnpm.ps1` wrapper, use `pnpm.cmd` for the same commands. There is currently no automated test suite or `test` script in the repository.

For Vercel deployment, configure `NEXT_PUBLIC_API_URL` in the target environment before building. The backend must be reachable by the deployment. Images are allowed from `via.placeholder.com`, `res.cloudinary.com`, `github.com`, and `images.unsplash.com`; update `images.remotePatterns` when introducing another image host. The root layout uses Google-hosted Geist fonts through `next/font/google`.

## Development workflow

1. Add a domain folder under `app/_modules/<feature>/`, creating only the layers that feature needs.
2. Define domain entities and Zod DTOs before wiring forms or requests.
3. Add a typed repository interface and an Axios implementation using the shared client.
4. Create stable query-key factories and query/mutation hooks. Include IDs and filters in keys, and invalidate each affected query prefix separately.
5. Build views using shared controls and validation components. Include loading, empty, error, and pending states.
6. Expose views through thin `app/(pages)/.../page.tsx` files, adding layouts/guards where appropriate and coordinating permissions with the backend.
7. Run `pnpm lint` and `pnpm build`, then exercise affected flows against a configured backend, including signed-out and non-admin behavior.

For framework changes, follow `AGENTS.md` and consult the version-specific guides under `node_modules/next/dist/docs/` before relying on older Next.js conventions.

## Screenshots

Screenshots have not been added yet. These filenames are suggested placeholders, not existing image assets. Capture the running application with non-sensitive demo data and replace these entries with images when available.

| Screen | Suggested screenshot file |
| --- | --- |
| Landing page | `docs/screenshots/landing.png` |
| Products page | `docs/screenshots/products.png` |
| Product details | `docs/screenshots/product-details.png` |
| Cart | `docs/screenshots/cart.png` |
| Login | `docs/screenshots/login.png` |
| Register | `docs/screenshots/register.png` |
| Dashboard | `docs/screenshots/dashboard.png` |

## Future improvements

Proposed next steps based on the current implementation:

- Complete dashboard management lists and connect product/category deletion; add user and payment administration screens.
- Apply consistent guards to account/dashboard routes and prevent non-admin content from rendering during redirects.
- Add email verification/resend and forgot/reset password flows with backend support.
- Integrate a real payment checkout session and success/cancellation handling with backend-verified payment state.
- Connect cart removal and quantity editing; render actual order status in history.
- Standardize query keys, list/detail cache updates, and logout cleanup across authenticated queries.
- Align create/update schemas, improve validation error paths, and remove schema casts where possible.
- Complete error handling across views, including the missing error-state return in product details.
- Add unit/integration and end-to-end tests for authentication, filtering, cart operations, and admin permissions, followed by CI checks.
- Replace starter/inconsistent metadata and improve product-specific metadata and canonical URLs.
- Add product search, sorting, a theme switcher, and an accessibility review.

## Author

**Mo'men Alswafiri** — creator of NovaCart.

- [GitHub](https://github.com/momen-x)
- [LinkedIn](https://www.linkedin.com/in/mo%E2%80%99men-alswafiri-8b6491346)
- [Email](mailto:moamenalswafiri@gmail.com)

## License

No license file is currently included in this repository.
