# jjbalisan.com: moving off Kinsta to cut hosting costs

**Recommendation:** Keep WordPress as the editor, but run it only on your laptop. Export the site as static HTML and serve it free from Cloudflare Pages. Hosting drops from about $350–420/yr on Kinsta to $0, and the only remaining cost is the domain (about $10–15/yr). Budget 3–5 hours for the move.

**Caveat:** The egress proxy in this environment blocked jjbalisan.com, so I could not inspect the live site. The public page list (Home, Now, Writings, Impossible List, Privacy Policy) comes from search-engine results [1], and I have assumed the site is a small WordPress site with no logins, shop, or members area. Complete step 1 below before committing to anything.

---

## 1. What you pay now

| Kinsta plan | Monthly billing | Annual billing |
|---|---|---|
| Single (1 site, 35k visits/mo, 20 GB) | $35/mo = $420/yr | $350/yr ($30/mo equivalent) |

Sources: [2], [3]. A personal site with about five pages uses a small fraction of this plan's allowance. Most of what Kinsta charges for (Google Cloud servers, Cloudflare Enterprise, 24/7 support, staging) is capacity you don't use.

## 2. Options compared

| Option | Annual cost (hosting only) | Keeps the WP editor? | Maintenance | Verdict |
|---|---|---|---|---|
| **A. Local WordPress, static export to Cloudflare Pages** | **$0** | Yes (on your laptop) | Very low: no public PHP or database to patch | **Recommended** |
| B. Rebuild in a static site generator (Astro/Hugo) on Cloudflare Pages or GitHub Pages | $0 | No (Markdown in git) | Lowest over time | Good if you would rather write in Markdown. Costs more hours upfront |
| C. WordPress.com Personal | about $48/yr [4] | Yes (hosted) | Low | Easiest managed option. Check plugin and theme limits first |
| D. Budget shared WordPress host (Hostinger, SiteGround) | $36–216/yr, and renewal prices are 3–6× the intro price [5] | Yes | Medium: still a public WP install to patch | Not worth it compared with A or C |
| E. Self-managed VPS (Hetzner etc.) | roughly €60–100/yr after Hetzner's 2026 price increases [6] | Yes | High: you run OS patches, backups, TLS, and security | Not recommended for a personal site |

### Why A is the best option
- **Cost:** Cloudflare Pages serves static files with unlimited requests on the free plan and allows 500 builds a month [7]. The Cloudflare free plan also includes DDoS protection and a CDN.
- **Security:** With no public WordPress, the usual attack routes (plugin vulnerabilities, brute-force logins on `wp-login.php`) no longer exist.
- **Workflow:** You write in the same WordPress admin. The only change is clicking "Generate" and uploading the result. The free Simply Static plugin does the export [8], and Simply Static and Cloudflare both publish walkthroughs for this setup [9], [10].
- **What you lose:** Anything that needs a server to run: comments, built-in contact forms, WordPress search, and scheduled posts. Each has a free substitute (see step 1).

## 3. Migration plan (Option A)

### Step 1: Inventory (30 min)
Log into WP admin on Kinsta and record:
- [ ] Plugins in use. Mark any that need a server: contact forms (Contact Form 7, WPForms), comments, search, newsletter sign-up, analytics.
- [ ] Whether comments are enabled.
- [ ] Where DNS for jjbalisan.com is hosted (Kinsta DNS, your registrar, or Cloudflare) and **where the domain is registered**.
- [ ] **Email records.** If you receive mail at @balisanyukasmith.com or @jjbalisan.com, write down all MX, SPF, DKIM, and DMARC records. Breaking email is the most serious risk in this migration.
- [ ] Kinsta billing: monthly or annual, and the renewal date.

Replacements for dynamic features:
| Feature | Free replacement |
|---|---|
| Contact form | Formspree / Basin free tiers, a Cloudflare Pages Function, or a plain `mailto:` link |
| Comments | Remove them, or embed Giscus (backed by GitHub Discussions) |
| Search | Remove it (the site has about five pages), or use Pagefind (static search) |
| Analytics | Cloudflare Web Analytics (free, no cookies) |

If the inventory turns up something that needs server-side WordPress (membership, e-commerce), switch to Option C.

### Step 2: Back up (15 min)
- In MyKinsta, create and download a full backup (files + database).
- Also run WP admin → Tools → Export → All content (XML) as a second copy.

### Step 3: Run WordPress locally (30–60 min)
- Install **LocalWP** (free) and import the Kinsta backup. Alternatively, create a blank site and use a migration plugin such as All-in-One WP Migration or Duplicator.
- Check that every page looks correct at the local URL.
- Delete unused plugins and themes to keep the export small.

### Step 4: Export to static (30 min)
- Install **Simply Static** in the local WordPress [8].
- Settings: set the replacement URL to **absolute URLs → `https://jjbalisan.com`** and the delivery method to **ZIP**.
- Generate, then download the ZIP.
- Unzip it and spot-check: open a few pages, and search the files for `localhost` or `.local` to catch URLs that were not rewritten.

### Step 5: Deploy to Cloudflare Pages (30 min)
- Create a free Cloudflare account. Go to Workers & Pages → Create → Pages → **Direct Upload** and upload the folder [9], [10].
- Test at the `*.pages.dev` preview URL: every nav link, images, the privacy policy, the 404 page, and mobile layout.
- Optional (makes later updates repeatable): put the export folder in a GitHub repo and connect it to Pages, or deploy from the command line with `npx wrangler pages deploy ./export`.

### Step 6: Switch the domain (30 min, then up to 24 h to propagate)
- Add jjbalisan.com to Cloudflare. Cloudflare scans the existing DNS records. **Compare the imported records with your Step 1 list, especially the mail records**, before changing nameservers.
- At your registrar, change the nameservers to Cloudflare's.
- In the Pages project, go to Custom domains, add `jjbalisan.com` and `www.jjbalisan.com`, and redirect one to the other.
- Optional: transfer the domain registration to Cloudflare Registrar, which charges the wholesale price with no markup.

### Step 7: Verify, then cancel Kinsta
- Wait at least a week. Check that the site loads over HTTPS, email still arrives, and Google Search Console shows no new crawl errors.
- Keep the Kinsta backup ZIP permanently.
- **Timing of the cancellation:** Kinsta refunds in full within the first 30 days of a plan [11]. Sources disagree about pro-rated refunds on annual plans after that: some say the unused portion is refunded pro-rata, others say it is not [3], [12]. Check the billing page in MyKinsta or [Kinsta's cancellation doc](https://kinsta.com/docs/billing/wordpress-hosting-plans/cancel-wordpress-plan/). If you are on annual billing with no pro-rata refund, set cancellation to take effect at renewal.

### Updating the site afterwards
1. Open LocalWP and edit or publish as usual.
2. Simply Static → Generate → download the ZIP.
3. Upload it to Cloudflare Pages, or push to the git repo.

This takes about 5 minutes. If it is annoying, Simply Static Pro ($99/yr) automates the push [8], but that erases most of the savings. Option B (Markdown in git with automatic deploys) is the cheaper way to get automation.

## 4. Cost summary

| | Year 1 | Following years |
|---|---|---|
| Kinsta Single (current) | $350–420 | $350–420 |
| Option A | domain only (~$10–15) | domain only |
| Option C | ~$48 + domain | ~$48 + domain |

Savings with A: about **$340–410 per year**.

## 5. Risks and rollback
- **Email outage from DNS mistakes.** The Step 1 record list plus the comparison in Step 6 prevents this.
- **Broken links or asset URLs in the export.** Checking the preview URL in Step 5 catches them before DNS changes.
- **Rollback:** Until Kinsta is cancelled, you can point the DNS records back to Kinsta and be restored within minutes to hours.
- **Laptop loss.** Your editable source now lives only on your machine. Keep the LocalWP site folder in a synced or backed-up location, or put the export in git.

## 6. Something you didn't ask about
Search results for the homepage still describe you as having "recently left Cohere" [1], so the About text predates your MIRI fellowship and your current role at the Nightingale Collective. Rebuilding locally is a convenient time to update it.

---

## References
[1] [Web search results for jjbalisan.com (homepage, /now, /writings, /impossible-list, /privacy-policy) - accessed 27 Sep 2026](https://jjbalisan.com/)
[2] [Kinsta - WordPress hosting plans](https://kinsta.com/pricing/)
[3] [Kinsta Pricing 2026: Real Cost of All 10 Plans - northiscale.com](https://northiscale.com/guides/kinsta-pricing)
[4] [WordPress.com Pricing 2026 - tooltester.com](https://www.tooltester.com/en/reviews/wordpress-com-review/pricing/)
[5] [Managed WordPress hosting in 2026: the renewal rate is the real price - DEV Community](https://dev.to/tanit365/managed-wordpress-hosting-in-2026-the-renewal-rate-is-the-real-price-ol0)
[6] [Hetzner Price Adjustment 15 June 2026 - Hetzner Docs](https://docs.hetzner.com/general/infrastructure-and-availability/price-adjustment/)
[7] [Cloudflare Pages Free Tier Limits & Pricing 2026 - DEV Community](https://dev.to/david_viejo_4d48fdfa7cfff/cloudflare-pages-free-tier-limits-pricing-2026-1f8f); [Cloudflare Workers pricing docs](https://developers.cloudflare.com/workers/platform/pricing/)
[8] [Simply Static - WordPress.org plugin page](https://wordpress.org/plugins/simply-static/)
[9] [How to Use Cloudflare Pages With WordPress - Simply Static](https://simplystatic.com/tutorials/cloudflare-pages-wordpress/)
[10] [Deploy a static WordPress site - Cloudflare Pages docs](https://developers.cloudflare.com/pages/how-to/deploy-a-wordpress-site/)
[11] [Kinsta Docs - Money-Back Guarantee](https://kinsta.com/docs/service-information/money-back-guarantee/)
[12] [What's Kinsta's Refund Policy? - Medium](https://medium.com/@serverguardian17/whats-kinsta-s-refund-policy-money-back-guarantee-details-14ce9a755451)

Created by Claude on 27th September 2026
