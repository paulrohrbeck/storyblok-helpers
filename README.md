# Storyblok notes, gotchas, links, etc.
Collecting some notes around the CMS Storyblok

## Problems & solutions

Problem: Next.js build took a long time, and individual pages took over 5 seconds to build
Solution: Set cache + increase rate limit in the `apiOptions`

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
