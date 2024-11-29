# Storyblok – Notes, gotchas, links, etc.
Collecting some notes around the CMS Storyblok

## Problems & solutions

### Slow API requests
**Problem:** Next.js build took a long time, and individual pages took over 5 seconds to build whereas direct API calls where super fast.

**Solution:** Set cache + increase [rate limit](https://www.storyblok.com/docs/api/content-delivery/v2/getting-started/rate-limit) in the `apiOptions`

```
storyblokInit({
  ...
  apiOptions: {
    cache: {
      clear: "auto",
      type: "memory",
    },
    rateLimit: 1000,
  },
})
```

### Preview not updating

**Problem:** Preview in the editor shows old content after saving. This happens with ISR and fallback blocking because new pages are only updated if the revalidate API is called or there's a timeout set.

**Solution:** Use draft mode so that getStaticProps() is called on every request. Configure your Storyblok preview URL to `<<full url>>/api/draft/?secret=<<some secret>>&slug=` the slug is automatically appended.

```
import { cookies, draftMode } from "next/headers"
import { redirect } from "next/navigation"

export async function GET(request) {
  const searchParams = request.nextUrl.searchParams
  const secret = searchParams.get("secret")
  const slug = searchParams.get("slug")

  // Check the secret and next parameters
  if (secret !== process.env.STORYBLOK_WEBHOOK_SECRET || !slug) {
    return new Response("Invalid token or missing slug.", { status: 401 })
  }

  // Enable Draft Mode by setting the cookie
  draftMode().enable()

  // Fix cookie for Storyblok iframe (https://www.storyblok.com/faq/next-js-preview-iframes)
  const byPassCookie = cookies().get("__prerender_bypass")
  cookies().set({
    name: "__prerender_bypass",
    value: byPassCookie?.value,
    httpOnly: true,
    path: "/",
    secure: true,
    sameSite: "none",
  })

  let redirectToSlug = `/${slug}`
  if (!redirectToSlug.endsWith("/")) {
    redirectToSlug += "/"
  }
  redirect(`${redirectToSlug}?${searchParams.toString()}`)
}
```
