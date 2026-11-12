# NetworkAI

A creative community app built with the Next.js App Router, React, TypeScript, Tailwind CSS, and shadcn/ui. Accounts and posts come from the deployed Go API on Google App Engine.

## Run locally

Use Node.js 22.13 or later.

```sh
npm ci
cp .env.example .env.local
npm run dev
```

Open [localhost:3000](http://localhost:3000). Create an account or sign in to search posts, publish media, and use AI Studio. Saved posts are stored per account in this browser. Caption and prompt drafts are stored per tab.

| Variable | Purpose |
| --- | --- |
| `NEXT_PUBLIC_API_BASE_URL` | Public Go API origin. Defaults to `https://networkai-513321.uw.r.appspot.com`. Set it before building the frontend. |
| `OPENAI_API_KEY` | Server runtime credential for AI Studio, with access to `gpt-image-2`. Keep it in the ignored `.env.local` or the hosting platform's runtime secret store. |
| `NETWORKAI_APP_ORIGIN` | Public HTTPS frontend origin, for example `https://networkai.example`. Set it at runtime when TLS terminates at a reverse proxy. |

## Go endpoints

| Operation | Request | Success response |
| --- | --- | --- |
| Register | `POST /users` with JSON `{ username, password }` | Empty body |
| Sign in | `POST /auth/login` with JSON `{ username, password }` | Plain-text JWT |
| Discover and search | `GET /posts/search` with optional `keywords` or `user` | Post array or `null` |
| Publish | `POST /posts` with multipart fields `message` and `media_file` | Empty body |
| Delete | `DELETE /posts/{postId}` | The client sends the documented request. The supplied Go router has this route commented out and the deployed service returns 404. |

Search, publish, and delete use `Authorization: Bearer <token>`. The Go search handler prioritizes `user` over `keywords`. My posts queries by user and filters captions locally. The frontend keeps the post visible if deletion fails. There is no post update route in the Go API.

The upload handler identifies media by filename extension. Supported extensions are JPEG, JPG, PNG, GIF, MOV, MP4, AVI, FLV, and WMV. Posts with an `unknown` media type still show their caption and author. The browser caps images at 10 MiB and videos at 30 MiB to leave room for multipart overhead within the [App Engine request limit](https://docs.cloud.google.com/appengine/docs/flexible/how-requests-are-handled).

## AI Studio

The browser calls the same-origin Next.js route `POST /api/ai/image` with a bearer token and multipart `prompt`, optional `image`, and optional `size`. The route checks the token through Go's `GET /posts/search` before calling OpenAI with the server credential. Generated and refined images are editable drafts until published via `POST /posts`.

## Build and test

```sh
npm run lint
npm run typecheck
npm test
npm run build
npx playwright install chromium
npm run test:e2e
```

Automated tests isolate external requests to verify the contracts and exercise UI behavior without modifying deployed data or incurring OpenAI charges. Run the built application with `npm start`. Deploy it on a Node-capable host so the image route can run server-side.
