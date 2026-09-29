# NovaCart

A portfolio-focused e-commerce frontend built with **Next.js App Router, React, and TypeScript**.

NovaCart uses an e-commerce scenario to demonstrate practical frontend engineering: authentication, product and category management, profile flows, a server-backed cart, checkout preparation, order history, API integration, form validation, image uploads, responsive UI, and deployment-oriented API configuration.

> **Portfolio scope:** NovaCart is designed as a technical showcase rather than a production commercial store. Some business flows are intentionally simplified or structured to demonstrate implementation concepts. In particular, the current payment experience is a portfolio demonstration and is **not presented as live production payment processing**.

This repository contains the **frontend**. A compatible external REST API is required for authentication, catalog data, uploads, cart/order data, and other server-backed behavior.

---

## Features

### Authentication & Profile

- Login and registration with validated forms, API mutations, notifications, and post-success navigation.
- Registration includes password confirmation and a terms agreement check.
- Logout through the backend API with cached current-user cleanup.
- Current-session lookup through `/users/me`; `401` is treated as an unauthenticated session.
- Profile details and first/last-name updates.
- Profile photo upload and removal.
- Supported profile image formats: JPEG, PNG, WebP, and GIF up to 5 MB.
- Password-change form with current-password and new-password confirmation.
- `GuestGuard` for login/register pages.
- `AdminGuard` on the dashboard index.

Current scope notes:

- Email-verification fields exist in user types, but verification/resend flows are not implemented in this frontend.
- Forgot/reset-password paths exist in constants, but the corresponding pages/API methods are not implemented.
- `AuthGuard` exists but is not currently mounted by a route.
- Client-side guards improve navigation/UX; the external API remains responsible for real authorization.

---

### Products & Categories

- Landing page with featured products and category discovery.
- Product catalog with category filtering and API-driven pagination.
- Eight products are requested per page.
- Category filtering is reflected through `?categoryId=...`.
- Product details with:
  - image
  - description
  - price
  - quantity selection
  - copy-link action
  - add-to-cart mutation
- Public category listing with links to filtered products.
- Product create/update forms with category selection and multipart image uploads.
- Category create/update forms with title and description.

Current scope notes:

- Product/category delete methods and hooks exist, but no visible delete UI currently invokes them.
- Dashboard product/category index pages are still placeholders.
- Product editing currently reuses the creation schema, so the form still requires an image file even though the update DTO supports partial changes.

---

### Cart, Checkout & Orders

- Add one item from a product card or a selected quantity from product details.
- Retrieve the current cart from the backend.
- Retrieve the cart badge count from the backend.
- Display order items, line totals, and the cart total.
- Collect shipping address, phone, and customer email before payment.
- Review shipping information and the order summary on `/cart/payment`.
- Complete the current order through `POST /orders`.
- Show success feedback and return to the storefront.
- View user order history with:
  - item images
  - quantities
  - totals
  - customer details

#### Payment Demonstration

The payment step is intentionally implemented as a **portfolio demonstration**.

The current frontend does **not** claim live Stripe payment processing. The payment screen includes Stripe-oriented UI and a demonstration video located at:

```text
public/assets/payment-ex.mp4
```

The current payment button completes the order through the backend order endpoint. This repository does not currently contain:

- Stripe SDK integration in the frontend
- checkout-session creation from the frontend
- a live Stripe Checkout redirect
- verified frontend evidence of production payment processing

This design is intentional for the current portfolio version of NovaCart: the checkout/payment step demonstrates where payment fits into the overall **cart → customer information → payment step → order completion** flow without presenting the project as a production-ready commercial store.

A commercial implementation would normally connect order completion to backend-verified payment state and add a complete success/cancellation/webhook-driven payment lifecycle.

Additional cart/order scope notes:

- Cart removal has an API method and hook, but the visible trash button currently has no handler.
- Cart quantity editing is not implemented.
- Order history currently displays a fixed `Delivered` badge instead of deriving the label from the order status.

---

### Dashboard & UI

- Collapsible dashboard sidebar.
- Product/category creation and editing routes.
- Responsive Tailwind layouts.
- Reusable shadcn/ui components.
- Lucide icons.
- Toast feedback.
- Shared skeleton/loading states for:
  - cards
  - product details
  - forms
  - cart
- Global loading page.
- Query retry panels.
- Offline/session error states.
- Empty states.
- Custom not-found page.
- Static About, Contact, FAQ, Privacy, and Terms pages.

Current scope notes:

- Dashboard overview and product/category list pages contain placeholder content.
- Users and Payments appear in the sidebar, but their dashboard routes are not implemented.
- User administration currently has repository/hooks only.
- Dark-mode styles exist, but there is no user-facing theme switcher yet.
- Error handling is not yet fully consistent across every view.

---

## Tech Stack

| Technology | Version / Range | Role |
|---|---:|---|
| Next.js | `16.3.5` | App Router, layouts, metadata, images/fonts, API rewrites |
| React / React DOM | `19.2.8` | Rendering and local UI state |
| TypeScript | `^5` | Strict typing and domain contracts |
| Tailwind CSS | `^4` | Styling and design tokens |
| shadcn | `^4.21.0` | Local UI components |
| Base UI | `^1.8.0` | Interactive UI primitives |
| React Hook Form | `^7.88.0` | Form state and field control |
| Hook Form Resolvers | `^5.9.1` | Zod integration |
| Zod | `^4.6.5` | Schemas and inferred DTO types |
| TanStack React Query | `^5.103.0` | Queries, mutations, and server-state cache |
| Axios | `^1.20.0` | HTTP client |
| React Toastify | `^11.1.0` | Notifications |
| Lucide React | `^1.46.0` | Icons |
| Class Variance Authority | `^0.7.1` | Component variants |
| pnpm | `10.28.2` | Package manager |

---

## Architecture

Routes are intentionally kept thin. They provide metadata, resolve dynamic parameters, and render feature views.

Business functionality is organized by domain inside `app/_modules`.

```text
App Router page
      ↓
Feature view
      ↓
Query / Mutation hook
      ↓
Repository
      ↓
Axios
      ↓
/api rewrite
      ↓
External REST API
```

### Main Layers

| Layer | Responsibility |
|---|---|
| `views/` | Screens and feature components; coordinate user interactions |
| `hooks/` | Queries, mutations, query keys, and cache updates |
| `repo/` | Typed API contracts and Axios implementations |
| `dto/` | Zod input schemas and inferred request/form types |
| `entities/` | Domain objects and API response structures |
| `adapters/` | Convert transport data into view-friendly shapes where needed |
| module `utils/` | Field definitions and feature-specific constants |
| `app/_utils/` | Shared Axios, API errors, network helpers, and formatting |
| `components/` | Shared UI primitives, guards, navigation, inputs, and skeletons |

For example, the order adapter flattens nested user/product data and converts prices to numbers before cart/order views consume them.

---

## Project Structure

```text
app/
├── (pages)/
│   ├── (auth)/
│   ├── about/
│   ├── cart/
│   │   └── payment/
│   ├── categories/
│   ├── contact/
│   ├── dashboard/
│   │   ├── categories/
│   │   └── products/
│   ├── faq/
│   ├── orders/
│   ├── privacy/
│   ├── products/
│   ├── profile/
│   └── terms/
│
├── _config/
├── _modules/
│   ├── auth/
│   ├── categories/
│   ├── dashboard/
│   ├── landing/
│   ├── order/
│   ├── order_item/
│   ├── products/
│   └── user/
│
├── _providers/
├── _types/
├── _utils/
├── globals.css
├── layout.tsx
├── loading.tsx
├── not-found.tsx
└── page.tsx

components/
├── guards/
├── icons/
├── inputs/
├── sharing/
├── skeletons/
└── ui/

lib/
public/assets/
```

---

## Routes

| Area | Paths |
|---|---|
| Storefront | `/`, `/products`, `/products/[id]/product-details`, `/categories` |
| Authentication | `/login`, `/register` |
| Account | `/profile`, `/profile/update-profile`, `/profile/update-password` |
| Shopping | `/cart`, `/cart/payment`, `/orders` |
| Dashboard | `/dashboard`, `/dashboard/products`, `/dashboard/categories` |
| Product Forms | `/dashboard/products/add`, `/dashboard/products/[id]/update` |
| Category Forms | `/dashboard/categories/add`, `/dashboard/categories/[id]/update` |
| Information | `/about`, `/contact`, `/faq`, `/privacy`, `/terms` |

---

## State Management & Data Fetching

TanStack React Query manages remote/server state. React local state handles transient UI concerns such as:

- selected quantity
- pagination
- sidebar visibility

There is no Redux or Zustand store.

Shared query defaults include:

- five-minute `staleTime`
- no window-focus refetching
- no automatic query retries
- `gcTime: 0`

Examples of query-key patterns:

```ts
productsQueryKeys.all
productsQueryKeys.filtered(1, 8, 2)
productsQueryKeys.one(42)
userQueryKeys.me()
orderItemQueryKeys.byOrderId(7)
```

The current code also contains areas where query-key consistency and authenticated cache cleanup can be improved. Those are treated as refactoring opportunities rather than hidden from the project documentation.

---

## API Communication

The shared Axios client uses the frontend origin:

```ts
axios.create({
  baseURL: "/api",
  withCredentials: true,
});
```

`next.config.ts` forwards `/api/:path*` to the configured external backend.

A browser call such as:

```text
/api/products
```

is rewritten to the backend base URL.

Authentication relies on backend-managed cookies. The frontend does not persist an authentication token in local storage and does not attach its own bearer token.

### API Areas Used

| Area | Example Endpoints |
|---|---|
| Authentication | `POST /users/auth/login`, `POST /users/auth/register`, `POST /users/auth/logout` |
| Profile | `GET /users/me`, user/profile update endpoints |
| Profile Images | photo upload/delete endpoints |
| Products | list/create/read/update/delete endpoints |
| Categories | list/create/read/update/delete endpoints |
| Orders | order, user-order, cart, and cart-count endpoints |
| Order Items | create/list/delete and order-scoped item endpoints |

Repository support does not imply a completed UI for every available endpoint.

---

## Forms & Validation

Forms use:

- React Hook Form
- `FormProvider`
- `Controller`
- `zodResolver`
- Zod schemas

Shared validation controls include:

- `ValidationInput`
- `ValidationSelect`
- `ValidationCheckbox`

DTO schemas cover areas including:

- authentication
- products
- categories
- profile changes
- password changes
- order details
- uploads

Server-side validation remains the responsibility of the API.

---

## Environment Variables

Create `.env.local`:

```env
NEXT_PUBLIC_API_URL=http://localhost:4000
```

Replace the value with the actual backend base URL and omit the trailing slash.

| Variable | Required | Purpose |
|---|---|---|
| `NEXT_PUBLIC_API_URL` | Yes | Absolute backend base URL used by the `/api` rewrite |

No Stripe, database, or Cloudinary secret is required by the checked-in frontend.

> `NEXT_PUBLIC_*` variables are public configuration and must never contain secret keys.

---

## Getting Started

### Requirements

- Node.js 20.9.0 or newer
- pnpm 10.28.2
- compatible backend API

### Install

```bash
git clone https://github.com/momen-x/novacart-frontend.git
cd novacart-frontend
pnpm install
```

### Run Development

```bash
pnpm dev
```

Open:

```text
http://localhost:3000
```

### Build

```bash
pnpm lint
pnpm build
pnpm start
```

For Vercel deployment, configure `NEXT_PUBLIC_API_URL` before building.

---

## Portfolio-Oriented Design Decisions

NovaCart intentionally focuses on demonstrating software-engineering concepts rather than reproducing every edge case or operational workflow expected from a real commercial store.

That means some scenarios are simplified, incomplete, or implemented mainly to make the feature visible within a portfolio demo. Examples include:

- the payment step being a demonstration rather than live payment processing
- incomplete dashboard-management views
- simplified cart behavior
- placeholder administrative sections
- some routes/repositories existing before their full UI is implemented

These choices should be read as the current project scope, not as a claim that NovaCart is a finished production commerce platform.

The project is primarily intended to demonstrate:

- modular frontend architecture
- typed API integration
- authentication/session consumption
- server-state management
- reusable form validation
- product/catalog flows
- cart/order workflows
- responsive UI implementation
- deployment-aware frontend/backend communication

---

## Future Improvements

- Complete dashboard product/category management lists.
- Connect delete actions to the visible UI.
- Add user and payment administration screens.
- Apply consistent guards to account/dashboard routes.
- Add email verification/resend and forgot/reset-password flows where supported by the backend.
- Replace the payment demonstration with a complete backend-verified payment flow if the project is evolved beyond portfolio scope.
- Connect cart removal and quantity editing.
- Render real order statuses.
- Standardize query keys and authenticated cache cleanup.
- Align create/update validation schemas.
- Improve error-state consistency.
- Add automated tests and CI checks.
- Improve product metadata and canonical URLs.
- Add search, sorting, theme switching, and accessibility review.

---

## Author

**Mo'men Alswafiri**

- GitHub: https://github.com/momen-x
- LinkedIn: https://www.linkedin.com/in/mo’men-alswafiri-8b6491346
- Email: moamenalswafiri@gmail.com

---

## License

No license file is currently included in this repository.
