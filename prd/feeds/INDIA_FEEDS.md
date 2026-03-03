# India-Focused Data Feeds

> A curated list of RSS feeds and data APIs for building an India-focused intelligence monitor.

---

## Already in World Monitor

| Source | Category | RSS URL |
|---|---|---|
| The Hindu | National News | `https://www.thehindu.com/news/national/feeder/default.rss` |
| Indian Express | National News | `https://indianexpress.com/section/india/feed/` |
| NDTV | National News | `https://feeds.feedburner.com/ndtvnews-top-stories` |
| India News Network | Foreign Policy | Google News proxy |
| Inc42 | Tech/Startups | `https://inc42.com/feed/` |
| YourStory | Startups | `https://yourstory.com/feed` |
| ORF Tech | Think Tank | Google News proxy |

---

## Recommended Additions

### Tier 1 — Wire Services & National News

| Source | RSS URL | Notes |
|---|---|---|
| PTI | `news.google.com/rss/search?q=site:ptinews.com` | India's official wire service |
| ANI | `news.google.com/rss/search?q=site:aninews.in` | Major wire service |
| Hindustan Times | `https://www.hindustantimes.com/feeds/rss/india-news/rssfeed.xml` | Top national daily |
| Times of India | `https://timesofindia.indiatimes.com/rssfeedstopstories.cms` | Largest English daily |
| Deccan Herald | `https://www.deccanherald.com/rss` | National daily |
| The Wire | `https://thewire.in/feed` | Independent investigative |
| Scroll.in | `https://scroll.in/feed` | Digital news |

### Tier 1 — Government & Official

| Source | RSS/API | Notes |
|---|---|---|
| PIB (Press Information Bureau) | `https://pib.gov.in/RssMain.aspx?ModId=6&Lang=1&Regid=3` | Official govt press releases |
| MEA (External Affairs) | `https://www.mea.gov.in/rss.xml` | Foreign policy & diplomacy |
| RBI | `https://rbi.org.in/Scripts/BS_PressReleaseDisplay.aspx?prid=rss` | Reserve Bank of India |
| SEBI | Google News proxy | Securities regulator |
| Parliament of India | Google News proxy | Legislative updates |
| MoD (Ministry of Defence) | Google News proxy | Defence press releases |

### Tier 2 — Business & Finance

| Source | RSS URL | Notes |
|---|---|---|
| Economic Times | `https://economictimes.indiatimes.com/rssfeedstopstories.cms` | Top business daily |
| Moneycontrol | `https://www.moneycontrol.com/rss/latestnews.xml` | Markets & finance |
| LiveMint | `https://www.livemint.com/rss/news` | Business news |
| Business Standard | `https://www.business-standard.com/rss/home_page_top_stories.rss` | Financial daily |
| NDTV Profit | Google News proxy | Business TV |

### Tier 2 — Defence & Security

| Source | RSS URL | Notes |
|---|---|---|
| Indian Defence News | `news.google.com/rss/search?q=India+defence+military` | Aggregated |
| MP-IDSA | Google News proxy | Manohar Parrikar Institute for Defence Studies |
| Bharat Shakti | `https://bharatshakti.in/feed/` | Defence & aerospace |
| India Strategic | Google News proxy | Defence analysis |
| Force Magazine | Google News proxy | Armed forces magazine |

### Tier 2 — Regional & Digital-First

| Source | RSS URL | Notes |
|---|---|---|
| The News Minute | `https://www.thenewsminute.com/feed` | South India focus |
| The Quint | `https://www.thequint.com/feed` | Digital-first national |
| Firstpost | `https://www.firstpost.com/rss/india.xml` | National |
| News18 | `https://www.news18.com/rss/india.xml` | National |
| India Today | `https://www.indiatoday.in/rss/home` | National magazine |
| Outlook India | Google News proxy | News magazine |

### Tier 2 — Tech & Startups

| Source | RSS URL | Notes |
|---|---|---|
| Entrackr | `https://entrackr.com/feed/` | Startup funding tracker |
| TechCircle | Google News proxy | VC/PE deals |
| MediaNama | `https://www.medianama.com/feed/` | India digital policy & telecom |

### Tier 3 — Think Tanks & Policy

| Source | RSS URL | Notes |
|---|---|---|
| ORF | `https://www.orfonline.org/feed` | Observer Research Foundation |
| Carnegie India | Google News proxy | Geopolitics & foreign policy |
| Takshashila Institution | Google News proxy | Public policy |
| Gateway House | Google News proxy | Indian foreign policy |
| CPR (Centre for Policy Research) | Google News proxy | Policy analysis |
| CEEW | Google News proxy | Energy, environment & water |

---

## India-Specific Data APIs

### Open / No Auth Required

| Source | URL | Data |
|---|---|---|
| data.gov.in | `https://data.gov.in/` | Open Government Data Platform (4,000+ datasets) |
| RBI DBIE | `https://dbie.rbi.org.in/` | Database on Indian Economy |
| IMD Weather | `https://mausam.imd.gov.in/` | India Meteorological Department |
| CPCB Air Quality | `https://app.cpcbccr.com/` | Real-time pollution monitoring |
| NDMA | `https://ndma.gov.in/` | National Disaster Management Authority |
| NSO/MOSPI | `https://mospi.gov.in/` | GDP, CPI, employment statistics |
| ISRO Bhuvan | `https://bhuvan.nrsc.gov.in/` | Satellite imagery & geospatial |

### Free with Registration

| Source | URL | Data |
|---|---|---|
| NSE APIs | `https://www.nseindia.com/` | Stock market data |
| UIDAI Aadhaar Stats | `https://uidai.gov.in/` | Digital identity statistics |
| India Stack | `https://indiastack.org/` | UPI, Aadhaar, DigiLocker stats |

---

## Quick-Start: Minimum Viable India Monitor

For a focused MVP, start with these **15 core feeds**:

### National News (4)
1. Times of India — `https://timesofindia.indiatimes.com/rssfeedstopstories.cms`
2. Hindustan Times — `https://www.hindustantimes.com/feeds/rss/india-news/rssfeed.xml`
3. The Hindu — `https://www.thehindu.com/news/national/feeder/default.rss`
4. NDTV — `https://feeds.feedburner.com/ndtvnews-top-stories`

### Business & Finance (3)
5. Economic Times — `https://economictimes.indiatimes.com/rssfeedstopstories.cms`
6. Moneycontrol — `https://www.moneycontrol.com/rss/latestnews.xml`
7. LiveMint — `https://www.livemint.com/rss/news`

### Government (2)
8. PIB — `https://pib.gov.in/RssMain.aspx?ModId=6&Lang=1&Regid=3`
9. MEA — `https://www.mea.gov.in/rss.xml`

### Investigative & Digital (3)
10. Indian Express — `https://indianexpress.com/section/india/feed/`
11. The Wire — `https://thewire.in/feed`
12. Scroll.in — `https://scroll.in/feed`

### Tech & Startups (2)
13. Inc42 — `https://inc42.com/feed/`
14. YourStory — `https://yourstory.com/feed`

### Think Tank (1)
15. ORF — `https://www.orfonline.org/feed`

---

## Notes

- **Google News proxy** = `https://news.google.com/rss/search?q=site:example.com&hl=en&gl=IN&ceid=IN:en` — useful when a site blocks datacenter IPs or has no RSS feed.
- **CORS** — All RSS feeds need server-side proxying for browser access. Use the existing `api/rss-proxy.js` pattern.
- **Hindi feeds** — Most sources above have Hindi editions. Add `lang: 'hi'` and use locale-boost for Hindi users. Examples:
  - BBC Hindi: `https://www.bbc.com/hindi/index.xml`
  - Navbharat Times: `https://navbharattimes.indiatimes.com/rssfeedsdefault.cms`
  - Dainik Jagran: Google News proxy with `hl=hi`
  - Amar Ujala: Google News proxy with `hl=hi`
