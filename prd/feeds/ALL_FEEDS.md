# World Monitor — Complete Feed Reference

> All RSS feeds, live data APIs, and intelligence sources used across World Monitor's four variants: **Full (Geopolitical)**, **Tech**, **Finance**, and **Happy**.

---

## Table of Contents

- [Geopolitical Variant (Full)](#geopolitical-variant-full)
- [Tech Variant](#tech-variant)
- [Finance Variant](#finance-variant)
- [Happy Variant](#happy-variant)
- [Intel Sources (Shared)](#intel-sources-shared)
- [Live Data APIs](#live-data-apis)
- [Telegram OSINT Channels](#telegram-osint-channels)
- [Live Webcams](#live-webcams)
- [Architecture Notes](#architecture-notes)

---

## Geopolitical Variant (Full)

### Politics

| Source | Tier | RSS URL |
|---|---|---|
| BBC World | 2 | `https://feeds.bbci.co.uk/news/world/rss.xml` |
| Guardian World | 2 | `https://www.theguardian.com/world/rss` |
| AP News | 1 | Google News proxy for `apnews.com` |
| Reuters World | 1 | Google News proxy for `reuters.com` |
| CNN World | 2 | Google News proxy for `cnn.com` |

### US News

| Source | Tier | RSS URL |
|---|---|---|
| Reuters US | 1 | Google News proxy for `reuters.com` |
| NPR News | 2 | `https://feeds.npr.org/1001/rss.xml` |
| PBS NewsHour | 2 | `https://www.pbs.org/newshour/feeds/rss/headlines` |
| ABC News | 2 | `https://feeds.abcnews.com/abcnews/topstories` |
| CBS News | 2 | `https://www.cbsnews.com/latest/rss/main` |
| NBC News | 2 | `https://feeds.nbcnews.com/nbcnews/public/news` |
| Wall Street Journal | 1 | `https://feeds.content.dowjones.io/public/rss/RSSUSnews` |
| Politico | 2 | `https://rss.politico.com/politics-news.xml` |
| The Hill | 3 | `https://thehill.com/news/feed` |
| Axios | 2 | `https://api.axios.com/feed/` |
| Fox News | 2 | `https://moxie.foxnews.com/google-publisher/us.xml` |

### Europe

| Source | Tier | Lang | RSS URL |
|---|---|---|---|
| France 24 | 2 | en/fr/es/ar | `https://www.france24.com/en/rss` (+ localized) |
| EuroNews | 2 | en/fr/de/it/es/pt/ru | `https://www.euronews.com/rss` (+ localized) |
| Le Monde | 2 | en/fr | `https://www.lemonde.fr/en/rss/une.xml` |
| DW News | 2 | en/de/es | `https://rss.dw.com/xml/rss-en-all` |
| El País | 2 | es | `https://feeds.elpais.com/mrss-s/pages/ep/site/elpais.com/portada` |
| El Mundo | 2 | es | `https://e00-elmundo.uecdn.es/elmundo/rss/portada.xml` |
| BBC Mundo | 2 | es | `https://www.bbc.com/mundo/index.xml` |
| Tagesschau | 1 | de | `https://www.tagesschau.de/xml/rss2/` |
| Bild | — | de | `https://www.bild.de/feed/alles.xml` |
| Der Spiegel | 2 | de | `https://www.spiegel.de/schlagzeilen/tops/index.rss` |
| Die Zeit | 2 | de | `https://newsfeed.zeit.de/index` |
| ANSA | 1 | it | `https://www.ansa.it/sito/notizie/topnews/topnews_rss.xml` |
| Corriere della Sera | 2 | it | `https://www.corriere.it/rss/homepage.xml` |
| Repubblica | 2 | it | `https://www.repubblica.it/rss/homepage/rss2.0.xml` |
| NOS Nieuws | 1 | nl | `https://feeds.nos.nl/nosnieuwsalgemeen` |
| NRC | 2 | nl | `https://www.nrc.nl/rss/` |
| De Telegraaf | 2 | nl | Google News proxy |
| SVT Nyheter | 1 | sv | `https://www.svt.se/nyheter/rss.xml` |
| Dagens Nyheter | 2 | sv | `https://www.dn.se/rss/` |
| Svenska Dagbladet | 2 | sv | `https://www.svd.se/feed/articles.rss` |
| BBC Türkçe | 2 | tr | `https://feeds.bbci.co.uk/turkce/rss.xml` |
| DW Turkish | 2 | tr | `https://rss.dw.com/xml/rss-tur-all` |
| Hürriyet | 2 | tr | `https://www.hurriyet.com.tr/rss/anasayfa` |
| TVN24 | 2 | pl | `https://tvn24.pl/swiat.xml` |
| Polsat News | 2 | pl | `https://www.polsatnews.pl/rss/wszystkie.xml` |
| Rzeczpospolita | 2 | pl | `https://www.rp.pl/rss_main` |
| Kathimerini | 2 | el | Google News proxy |
| Naftemporiki | 2 | el | `https://www.naftemporiki.gr/feed/` |
| in.gr | 3 | el | `https://www.in.gr/feed/` |
| iefimerida | 3 | el | `https://www.iefimerida.gr/rss.xml` |
| Proto Thema | 3 | el | Google News proxy |
| BBC Russian | 2 | ru | `https://feeds.bbci.co.uk/russian/rss.xml` |
| Meduza | 2 | ru | `https://meduza.io/rss/all` |
| Novaya Gazeta Europe | 2 | ru | `https://novayagazeta.eu/feed/rss` |
| TASS | 3 | — | Google News proxy |
| RT | 3 | — | `https://www.rt.com/rss/` |
| RT Russia | 3 | — | `https://www.rt.com/rss/russia/` |
| Kyiv Independent | — | — | Google News proxy |
| Moscow Times | — | — | `https://www.themoscowtimes.com/rss/news` |

### Middle East

| Source | Tier | Lang | RSS URL |
|---|---|---|---|
| BBC Middle East | 2 | — | `https://feeds.bbci.co.uk/news/world/middle_east/rss.xml` |
| Al Jazeera | 2 | en/ar | `https://www.aljazeera.com/xml/rss/all.xml` |
| Al Arabiya | — | en/ar | Google News proxy (EN) / Direct (AR) |
| Guardian ME | 2 | — | `https://www.theguardian.com/world/middleeast/rss` |
| BBC Persian | 2 | — | `http://feeds.bbci.co.uk/persian/tv-and-radio-37434376/rss.xml` |
| Iran International | 3 | — | Google News proxy |
| Fars News | 3 | — | Google News proxy |
| Haaretz | — | — | Google News proxy |
| Arab News | — | — | Google News proxy |
| The National | 2 | — | Google News proxy |
| Oman Observer | — | — | `https://www.omanobserver.om/rssFeed/1` |
| Asharq Business | — | — | `https://asharqbusiness.com/rss.xml` |
| Asharq News | — | ar | `https://asharq.com/snapchat/rss.xml` |

### Africa

| Source | Tier | Lang | RSS URL |
|---|---|---|---|
| Africa News | — | — | Google News proxy |
| Sahel Crisis | — | — | Google News proxy |
| News24 | — | — | `https://feeds.news24.com/articles/news24/TopStories/rss` |
| BBC Africa | — | — | `https://feeds.bbci.co.uk/news/world/africa/rss.xml` |
| Jeune Afrique | — | fr | `https://www.jeuneafrique.com/feed/` |
| Africanews | — | en/fr | `https://www.africanews.com/feed/rss` |
| BBC Afrique | — | fr | `https://www.bbc.com/afrique/index.xml` |
| Premium Times | 2 | — | `https://www.premiumtimesng.com/feed` |
| Vanguard Nigeria | 2 | — | `https://www.vanguardngr.com/feed/` |
| Channels TV | 2 | — | `https://www.channelstv.com/feed/` |
| Daily Trust | 3 | — | `https://dailytrust.com/feed/` |
| ThisDay | 2 | — | `https://www.thisdaylive.com/feed` |

### Latin America

| Source | Tier | Lang | RSS URL |
|---|---|---|---|
| Latin America | — | — | Google News proxy |
| BBC Latin America | — | — | `https://feeds.bbci.co.uk/news/world/latin_america/rss.xml` |
| Reuters LatAm | — | — | Google News proxy |
| Guardian Americas | — | — | `https://www.theguardian.com/world/americas/rss` |
| Clarín | — | es | `https://www.clarin.com/rss/lo-ultimo/` |
| O Globo | — | pt | Google News proxy |
| Folha de S.Paulo | — | pt | `https://feeds.folha.uol.com.br/emcimadahora/rss091.xml` |
| Brasil Paralelo | 2 | pt | `https://www.brasilparalelo.com.br/noticias/rss.xml` |
| El Tiempo | — | es | `https://www.eltiempo.com/rss/mundo_latinoamerica.xml` |
| La Silla Vacía | 3 | — | `https://www.lasillavacia.com/rss` |
| Primicias | — | es | `https://www.primicias.ec/feed/` |
| Infobae Americas | — | es | `https://www.infobae.com/feeds/rss/` |
| El Universo | — | es | `https://www.eluniverso.com/arc/outboundfeeds/rss/...` |
| Mexico News Daily | — | — | `https://mexiconewsdaily.com/feed/` |
| Mexico Security | — | — | Google News proxy |
| AP Mexico | — | — | Google News proxy |
| InSight Crime | — | — | `https://insightcrime.org/feed/` |
| France 24 LatAm | — | — | `https://www.france24.com/en/americas/rss` |

### Asia-Pacific

| Source | Tier | Lang | RSS URL |
|---|---|---|---|
| Asia News | — | — | Google News proxy |
| BBC Asia | — | — | `https://feeds.bbci.co.uk/news/world/asia/rss.xml` |
| The Diplomat | 3 | — | `https://thediplomat.com/feed/` |
| South China Morning Post | — | — | `https://www.scmp.com/rss/91/feed/` (Railway proxy) |
| Reuters Asia | — | — | Google News proxy |
| Xinhua | 3 | — | Google News proxy |
| Japan Today | — | — | `https://japantoday.com/feed/atom` |
| Nikkei Asia | 2 | — | Google News proxy |
| Asahi Shimbun | — | ja | `https://www.asahi.com/rss/asahi/newsheadlines.rdf` |
| The Hindu | — | — | `https://www.thehindu.com/news/national/feeder/default.rss` |
| Indian Express | — | — | `https://indianexpress.com/section/india/feed/` |
| NDTV | — | — | `https://feeds.feedburner.com/ndtvnews-top-stories` |
| India News Network | — | — | Google News proxy |
| CNA | — | — | `https://www.channelnewsasia.com/api/v1/rss-outbound-feed?_format=xml` |
| MIIT (China) | 1 | zh | Google News proxy |
| MOFCOM (China) | 1 | zh | Google News proxy |
| Bangkok Post | 2 | th | Google News proxy |
| Thai PBS | 2 | th | Google News proxy |
| VnExpress | 2 | vi | `https://vnexpress.net/rss/tin-moi-nhat.rss` |
| Tuoi Tre News | 2 | vi | `https://tuoitrenews.vn/rss` |
| Yonhap News | 2 | ko | `https://www.yonhapnewstv.co.kr/browse/feed/` |
| Chosun Ilbo | 2 | ko | `https://www.chosun.com/arc/outboundfeeds/rss/?outputType=xml` |
| ABC News Australia | 2 | — | `https://www.abc.net.au/news/feed/2942460/rss.xml` |
| Guardian Australia | 2 | — | `https://www.theguardian.com/australia-news/rss` |
| Island Times (Palau) | — | — | `https://islandtimes.org/feed/` |

### Tech

| Source | Tier | RSS URL |
|---|---|---|
| Hacker News | 4 | `https://hnrss.org/frontpage` |
| Ars Technica | 3 | `https://feeds.arstechnica.com/arstechnica/technology-lab` |
| The Verge | 4 | `https://www.theverge.com/rss/index.xml` |
| MIT Tech Review | 3 | `https://www.technologyreview.com/feed/` |

### AI

| Source | Tier | RSS URL |
|---|---|---|
| AI News | 4 | Google News proxy (OpenAI, Anthropic, etc.) |
| VentureBeat AI | 4 | `https://venturebeat.com/category/ai/feed/` |
| The Verge AI | 4 | `https://www.theverge.com/rss/ai-artificial-intelligence/index.xml` |
| MIT Tech Review AI | — | `https://www.technologyreview.com/topic/artificial-intelligence/feed` |
| ArXiv AI | 4 | `https://export.arxiv.org/rss/cs.AI` |

### Finance

| Source | Tier | RSS URL |
|---|---|---|
| CNBC | 2 | `https://www.cnbc.com/id/100003114/device/rss/rss.html` |
| MarketWatch | 2 | Google News proxy |
| Yahoo Finance | 4 | `https://finance.yahoo.com/news/rssindex` |
| Financial Times | 2 | `https://www.ft.com/rss/home` |
| Reuters Business | 1 | Google News proxy |

### Government & Official

| Source | Tier | RSS URL |
|---|---|---|
| White House | 1 | Google News proxy |
| State Dept | 1 | Google News proxy |
| Pentagon | 1 | Google News proxy |
| Treasury | 2 | Google News proxy |
| DOJ | 2 | Google News proxy |
| Federal Reserve | 3 | `https://www.federalreserve.gov/feeds/press_all.xml` |
| SEC | 3 | `https://www.sec.gov/news/pressreleases.rss` |
| CDC | 2 | Google News proxy |
| FEMA | 2 | Google News proxy |
| DHS | 2 | Google News proxy |
| UN News | 1 | `https://news.un.org/feed/subscribe/en/news/all/rss.xml` (Railway) |
| CISA | 1 | `https://www.cisa.gov/cybersecurity-advisories/all.xml` (Railway) |

### Think Tanks

| Source | Tier | RSS URL |
|---|---|---|
| Foreign Policy | 3 | `https://foreignpolicy.com/feed/` |
| Atlantic Council | 3 | `https://www.atlanticcouncil.org/feed/` (Railway) |
| Foreign Affairs | 3 | `https://www.foreignaffairs.com/rss.xml` |
| CSIS | 3 | Google News proxy |
| RAND | 3 | Google News proxy |
| Brookings | 3 | Google News proxy |
| Carnegie | 3 | Google News proxy |
| War on the Rocks | 2 | `https://warontherocks.com/feed` |
| AEI | 3 | `https://www.aei.org/feed/` |
| Responsible Statecraft | 3 | `https://responsiblestatecraft.org/feed/` |
| RUSI | 2 | Google News proxy |
| FPRI | 3 | `https://www.fpri.org/feed/` |
| Jamestown | 3 | `https://jamestown.org/feed/` |

### Crisis & International Orgs

| Source | Tier | RSS URL |
|---|---|---|
| CrisisWatch | 3 | `https://www.crisisgroup.org/rss` |
| IAEA | 1 | `https://www.iaea.org/feeds/topnews` |
| WHO | 1 | `https://www.who.int/rss-feeds/news-english.xml` |
| UNHCR | 1 | Google News proxy |

### Energy

| Source | Tier | RSS URL |
|---|---|---|
| Oil & Gas | — | Google News proxy |
| Nuclear Energy | — | Google News proxy |
| Reuters Energy | — | Google News proxy |
| Mining & Resources | — | Google News proxy |

### Layoffs

| Source | Tier | RSS URL |
|---|---|---|
| Layoffs.fyi | 3 | Google News proxy |
| TechCrunch Layoffs | 4 | `https://techcrunch.com/tag/layoffs/feed/` |
| Layoffs News | 4 | Google News proxy |

---

## Tech Variant

### Core Tech

| Source | RSS URL |
|---|---|
| TechCrunch | `https://techcrunch.com/feed/` |
| The Verge | `https://www.theverge.com/rss/index.xml` |
| Ars Technica | `https://feeds.arstechnica.com/arstechnica/technology-lab` |
| Hacker News | `https://hnrss.org/frontpage` |
| MIT Tech Review | `https://www.technologyreview.com/feed/` |
| ZDNet | `https://www.zdnet.com/news/rss.xml` |
| TechMeme | `https://www.techmeme.com/feed.xml` |
| Engadget | `https://www.engadget.com/rss.xml` |
| Fast Company | `https://feeds.feedburner.com/fastcompany/headlines` |

### AI & ML

| Source | RSS URL |
|---|---|
| AI News | Google News proxy |
| VentureBeat AI | `https://venturebeat.com/category/ai/feed/` |
| The Verge AI | `https://www.theverge.com/rss/ai-artificial-intelligence/index.xml` |
| MIT Tech Review AI | `https://www.technologyreview.com/topic/artificial-intelligence/feed` |
| MIT Research | `https://news.mit.edu/rss/research` |
| ArXiv AI | `https://export.arxiv.org/rss/cs.AI` |
| ArXiv ML | `https://export.arxiv.org/rss/cs.LG` |
| AI Weekly | Google News proxy |
| Anthropic News | Google News proxy |
| OpenAI News | Google News proxy |

### Startups

| Source | RSS URL |
|---|---|
| TechCrunch Startups | `https://techcrunch.com/category/startups/feed/` |
| VentureBeat | `https://venturebeat.com/feed/` |
| Crunchbase News | `https://news.crunchbase.com/feed/` |
| SaaStr | `https://www.saastr.com/feed/` |
| AngelList News | Google News proxy |
| TechCrunch Venture | `https://techcrunch.com/category/venture/feed/` |
| The Information | Google News proxy |
| Fortune Term Sheet | Google News proxy |
| PitchBook News | Google News proxy |
| CB Insights | `https://www.cbinsights.com/research/feed/` |

### VC Blogs & Newsletters

| Source | RSS URL |
|---|---|
| Y Combinator Blog | `https://www.ycombinator.com/blog/rss/` |
| a16z Blog | Google News proxy |
| Sequoia Blog | Google News proxy |
| Paul Graham Essays | Google News proxy |
| VC Insights | Google News proxy |
| Lenny's Newsletter | `https://www.lennysnewsletter.com/feed` |
| Stratechery | `https://stratechery.com/feed/` |
| FwdStart Newsletter | `/api/fwdstart` (internal) |

### Regional Startups

| Region | Sources |
|---|---|
| **Europe** | EU Startups, Tech.eu, Sifted, The Next Web |
| **Asia General** | Tech in Asia, KrASIA, SEA Startups, Asia VC News |
| **China** | China Startups, 36Kr English, China Tech Giants |
| **Japan** | Japan Startups, Japan Tech News, Nikkei Tech |
| **Korea** | Korea Tech News, Korea Startups |
| **India** | Inc42, YourStory, India Startups, India Tech News |
| **Southeast Asia** | SEA Tech News, Vietnam Tech, Indonesia Tech |
| **Taiwan** | Taiwan Tech |
| **Latin America** | LAVCA, LATAM Startups, Brazil Tech, FinTech LATAM |
| **Africa** | TechCabal, Africa Startups, Africa Tech News |
| **Middle East** | MENA Startups, MENA Tech News |

### GitHub & Developer

| Source | RSS URL |
|---|---|
| GitHub Blog | `https://github.blog/feed/` |
| GitHub Trending | `https://mshibanami.github.io/GitHubTrendingRSS/daily/all.xml` |
| Show HN | `https://hnrss.org/show` |
| YC Launches | Google News proxy |
| Dev Events | Google News proxy |
| Open Source News | Google News proxy |

### Security

| Source | RSS URL |
|---|---|
| Krebs Security | `https://krebsonsecurity.com/feed/` |
| The Hacker News | `https://feeds.feedburner.com/TheHackersNews` |
| Dark Reading | `https://www.darkreading.com/rss.xml` |
| Schneier | `https://www.schneier.com/feed/` |

### Hardware & Cloud

| Source | RSS URL |
|---|---|
| Tom's Hardware | `https://www.tomshardware.com/feeds/all` |
| SemiAnalysis | Google News proxy |
| Semiconductor News | Google News proxy |
| InfoQ | `https://feed.infoq.com/` |
| The New Stack | `https://thenewstack.io/feed/` |
| DevOps.com | `https://devops.com/feed/` |
| Dev.to | `https://dev.to/feed` |
| Lobsters | `https://lobste.rs/rss` |
| Changelog | `https://changelog.com/feed` |

### Tech Policy & Think Tanks

| Source | RSS URL |
|---|---|
| Politico Tech | `https://rss.politico.com/technology.xml` |
| AI Regulation | Google News proxy |
| EFF News | Google News proxy |
| EU Digital Policy | Google News proxy |
| Stanford HAI | Google News proxy |
| AI Now Institute | Google News proxy |
| OECD Digital | Google News proxy |
| DigiChina | Google News proxy |
| + 15 more regional think tanks | Various Google News proxies |

### Podcasts & Media

| Source | RSS URL |
|---|---|
| Acquired Episodes | Google News proxy |
| All-In Podcast | Google News proxy |
| a16z Insights | Google News proxy |
| This Week in Startups | Google News proxy |
| 20VC Episodes | `https://rss.libsyn.com/shows/61840/destinations/240976.xml` |
| Lex Fridman Tech | Google News proxy |
| Hard Fork (NYT) | Google News proxy |
| Pivot Podcast | `https://feeds.megaphone.fm/pivot` |
| Masters of Scale | `https://rss.art19.com/masters-of-scale` |
| + 8 more | Various |

### Other (Tech Variant)

| Category | Sources |
|---|---|
| **IPO** | IPO News, Renaissance IPO, Tech IPO News |
| **Funding** | SEC Filings, VC News, Seed & Pre-Seed, Startup Funding |
| **Product Hunt** | `https://www.producthunt.com/feed` |
| **Outages** | AWS Status, Cloud Outages |
| **Unicorns** | Unicorn News, CB Insights Unicorn, Decacorn News |
| **Accelerators** | Techstars, 500 Global, Demo Day, Startup School |

---

## Finance Variant

### Markets

| Source | RSS URL |
|---|---|
| CNBC | `https://www.cnbc.com/id/100003114/device/rss/rss.html` |
| MarketWatch | Google News proxy |
| Yahoo Finance | `https://finance.yahoo.com/rss/topstories` |
| Seeking Alpha | `https://seekingalpha.com/market_currents.xml` |
| Reuters Markets | Google News proxy |
| Bloomberg Markets | Google News proxy |
| Investing.com News | Google News proxy |

### Forex

| Source | RSS URL |
|---|---|
| Forex News | Google News proxy |
| Dollar Watch | Google News proxy |
| Central Bank Rates | Google News proxy |

### Bonds

| Source | RSS URL |
|---|---|
| Bond Market | Google News proxy |
| Treasury Watch | Google News proxy |
| Corporate Bonds | Google News proxy |

### Commodities

| Source | RSS URL |
|---|---|
| Oil & Gas | Google News proxy |
| Gold & Metals | Google News proxy |
| Agriculture | Google News proxy |
| Commodity Trading | Google News proxy |

### Crypto

| Source | RSS URL |
|---|---|
| CoinDesk | `https://www.coindesk.com/arc/outboundfeeds/rss/` |
| Cointelegraph | `https://cointelegraph.com/rss` |
| The Block | Google News proxy |
| Crypto News | Google News proxy |
| DeFi News | Google News proxy |

### Central Banks

| Source | RSS URL |
|---|---|
| Federal Reserve | `https://www.federalreserve.gov/feeds/press_all.xml` |
| ECB Watch | Google News proxy |
| BoJ Watch | Google News proxy |
| BoE Watch | Google News proxy |
| PBoC Watch | Google News proxy |
| Global Central Banks | Google News proxy |

### Economic Data

| Source | RSS URL |
|---|---|
| Economic Data | Google News proxy (CPI, GDP, PMI, etc.) |
| Trade & Tariffs | Google News proxy |
| Housing Market | Google News proxy |

### IPO & Corporate

| Source | RSS URL |
|---|---|
| IPO News | Google News proxy |
| Earnings Reports | Google News proxy |
| M&A News | Google News proxy |

### Derivatives, FinTech, Regulation, Institutional, Analysis

| Category | Count | Sources |
|---|---|---|
| **Derivatives** | 2 | Options Market, Futures Trading |
| **FinTech** | 3 | Fintech News, Trading Tech, Blockchain Finance |
| **Regulation** | 4 | SEC, Financial Regulation, Banking Rules, Crypto Regulation |
| **Institutional** | 3 | Hedge Fund News, Private Equity, Sovereign Wealth |
| **Analysis** | 3 | Market Outlook, Risk & Volatility, Bank Research |

### GCC/MENA Finance

| Source | RSS URL |
|---|---|
| Arabian Business | Google News proxy |
| The National | Google News proxy |
| Arab News | Google News proxy |
| Gulf FDI | Google News proxy (PIF, DP World, Mubadala, ADNOC) |
| Gulf Investments | Google News proxy |
| Vision 2030 | Google News proxy |

---

## Happy Variant

### Positive News

| Source | RSS URL |
|---|---|
| Good News Network | `https://www.goodnewsnetwork.org/feed/` |
| Positive.News | `https://www.positive.news/feed/` |
| Reasons to be Cheerful | `https://reasonstobecheerful.world/feed/` |
| Optimist Daily | `https://www.optimistdaily.com/feed/` |
| Upworthy | `https://www.upworthy.com/feed/` |
| DailyGood | `https://www.dailygood.org/feed` |
| Good Good Good | `https://www.goodgoodgood.co/articles/rss.xml` |
| GOOD Magazine | `https://www.good.is/feed/` |
| Sunny Skyz | `https://www.sunnyskyz.com/rss_tebow.php` |
| The Better India | `https://thebetterindia.com/feed/` |

### Science & Nature

| Source | RSS URL |
|---|---|
| GNN Science | `https://www.goodnewsnetwork.org/category/news/science/feed/` |
| ScienceDaily | `https://www.sciencedaily.com/rss/all.xml` |
| Nature News | `https://feeds.nature.com/nature/rss/current` |
| Live Science | `https://www.livescience.com/feeds.xml` |
| New Scientist | `https://www.newscientist.com/feed/home/` |
| Singularity Hub | `https://singularityhub.com/feed/` |
| Human Progress | `https://humanprogress.org/feed/` |
| Greater Good (Berkeley) | `https://greatergood.berkeley.edu/site/rss/articles` |
| GNN Animals | `https://www.goodnewsnetwork.org/category/news/animals/feed/` |
| GNN Health | `https://www.goodnewsnetwork.org/category/news/health/feed/` |
| GNN Heroes | `https://www.goodnewsnetwork.org/category/news/inspiring/feed/` |

---

## Intel Sources (Shared)

Loaded on the geopolitical variant as a dedicated intelligence panel.

### Defence & Security

| Source | Tier | Type | RSS URL |
|---|---|---|---|
| Defense One | 3 | defense | `https://www.defenseone.com/rss/all/` |
| Breaking Defense | 3 | defense | `https://breakingdefense.com/feed/` |
| The War Zone | 3 | defense | `https://www.twz.com/feed` |
| Defense News | 3 | defense | `https://www.defensenews.com/arc/outboundfeeds/rss/...` |
| Janes | 3 | defense | Google News proxy |
| Military Times | 2 | defense | `https://www.militarytimes.com/arc/outboundfeeds/rss/...` |
| Task & Purpose | 3 | defense | `https://taskandpurpose.com/feed/` |
| USNI News | 2 | defense | `https://news.usni.org/feed` |
| gCaptain | 3 | defense | `https://gcaptain.com/feed/` |
| Oryx OSINT | 2 | defense | `https://www.oryxspioenkop.com/feeds/posts/default?alt=rss` |
| UK MOD | 1 | defense | `https://www.gov.uk/government/organisations/ministry-of-defence.atom` |
| CSIS | — | defense | Google News proxy |

### International Relations

| Source | Type | RSS URL |
|---|---|---|
| Chatham House | intl | Google News proxy |
| ECFR | intl | Google News proxy |
| Foreign Policy | intl | `https://foreignpolicy.com/feed/` |
| Foreign Affairs | intl | `https://www.foreignaffairs.com/rss.xml` |
| Atlantic Council | intl | `https://www.atlanticcouncil.org/feed/` (Railway) |
| Middle East Institute | intl | Google News proxy |

### Research & Think Tanks

| Source | Type | RSS URL |
|---|---|---|
| RAND | research | Google News proxy |
| Brookings | research | Google News proxy |
| Carnegie | research | Google News proxy |
| FAS | research | Google News proxy |
| NTI | research | Google News proxy |
| RUSI | research | Google News proxy |
| Wilson Center | research | Google News proxy |
| GMF | research | Google News proxy |
| Stimson Center | research | `https://www.stimson.org/feed/` |
| CNAS | research | Google News proxy |
| Lowy Institute | research | Google News proxy |

### Nuclear & Arms Control

| Source | Type | RSS URL |
|---|---|---|
| Arms Control Assn | nuclear | Google News proxy |
| Bulletin of Atomic Scientists | nuclear | Google News proxy |

### OSINT & Cyber

| Source | Type | RSS URL |
|---|---|---|
| Bellingcat | osint | Google News proxy |
| Krebs Security | cyber | `https://krebsonsecurity.com/feed/` |
| Ransomware.live | cyber | `https://www.ransomware.live/rss.xml` |

### Economic & Food Security

| Source | Type | RSS URL |
|---|---|---|
| FAO News | economic | `https://www.fao.org/feeds/fao-newsroom-rss` |
| FAO GIEWS | economic | Google News proxy |
| EU ISS | intl | Google News proxy |

---

## Live Data APIs

| Data Source | Provider | Auth Required | Env Variable |
|---|---|---|---|
| **AIS Vessel Tracking** | AISStream.io (WebSocket) | API Key | `AISSTREAM_API_KEY` |
| **Aircraft Positions** | OpenSky Network | OAuth2 (free) | `OPENSKY_CLIENT_ID` / `SECRET` |
| **Aircraft Enrichment** | Wingbits | API Key | `WINGBITS_API_KEY` |
| **Stock Quotes** | Finnhub | API Key (free tier) | `FINNHUB_API_KEY` |
| **Energy Data** | U.S. EIA | API Key (free) | `EIA_API_KEY` |
| **Economic Data** | FRED (Federal Reserve) | API Key (free) | `FRED_API_KEY` |
| **Conflict Events** | ACLED | Access Token (free/research) | `ACLED_ACCESS_TOKEN` |
| **Conflict Data** | UCDP (Uppsala) | Access Token | `UCDP_ACCESS_TOKEN` |
| **Internet Outages** | Cloudflare Radar | API Token (free) | `CLOUDFLARE_API_TOKEN` |
| **Satellite Fires** | NASA FIRMS | API Key (free) | `NASA_FIRMS_API_KEY` |
| **AI Summarization** | Groq (14,400 req/day free) | API Key | `GROQ_API_KEY` |
| **AI Fallback** | OpenRouter (50 req/day free) | API Key | `OPENROUTER_API_KEY` |
| **Cache** | Upstash Redis | Token (free tier) | `UPSTASH_REDIS_REST_URL` |
| **Prediction Markets** | Polymarket | None | — |
| **GPS Interference** | gpsjam.org | None (scraped) | — |
| **OREF Rocket Alerts** | oref.org.il | None (residential proxy) | — |
| **Earthquake Data** | USGS | None (public domain) | — |
| **Humanitarian Data** | UN OCHA HAPI | None (CC BY 4.0) | — |
| **Climate Anomalies** | Open-Meteo (ERA5) | None | — |
| **Displacement Data** | UNHCR | None (CC BY 4.0) | — |
| **Travel Advisories** | US State, AU DFAT, UK FCDO, NZ MFAT | None | — |
| **US Embassy Alerts** | 13 country-specific feeds | None | — |
| **Health Alerts** | CDC, ECDC, WHO | None | — |

---

## Telegram OSINT Channels

26 curated channels polled via MTProto relay:

| Channel | Category |
|---|---|
| Aurora Intel | OSINT |
| BNO News | Breaking |
| OSINTdefender | OSINT |
| DeepState | Conflict |
| LiveUAMap | Conflict |
| + 21 more | Various (breaking/conflict/alerts/osint/politics/middleeast) |

**Auth**: Telegram API ID + Hash + StringSession (`TELEGRAM_API_ID`, `TELEGRAM_API_HASH`, `TELEGRAM_SESSION`)

---

## Live Webcams

22 YouTube live streams from geopolitical hotspots across 5 regions:

| Region | Coverage |
|---|---|
| Iran / Attacks | Tehran, Tel Aviv, Jerusalem (dedicated 2×2 grid) |
| Middle East | Various hotspots |
| Europe | Various |
| Americas | Various |
| Asia-Pacific | Various |

---

## Architecture Notes

### RSS Proxy

All RSS feeds are fetched through `api/rss-proxy.js` — a domain-allowlisted Vercel Edge Function that:
- Prevents CORS issues
- Adds circuit breakers with 5-minute cooldowns
- Falls back to Railway relay for blocked feeds

### Google News Proxy Pattern

For sites that block cloud IPs, the project uses:
```
https://news.google.com/rss/search?q=site:example.com&hl=en-US&gl=US&ceid=US:en
```
This returns Google News results filtered to a specific domain.

### Server-Side Aggregation

A single `listFeedDigest` RPC call:
- Fetches all feeds server-side (batched at 20 concurrent, 8s per-feed timeout, 25s overall deadline)
- Caches the categorized digest in Redis for 15 minutes
- Individual feed results are cached for 10 minutes
- Reduces Vercel Edge invocations by ~95%

### Source Tier System

| Tier | Description | Examples |
|---|---|---|
| 1 | Wire services — fastest, most reliable | Reuters, AP, AFP, Bloomberg |
| 2 | Major outlets — high-quality journalism | BBC, Guardian, NPR, CNBC |
| 3 | Specialty sources — domain expertise | Defense One, Bellingcat, CSIS |
| 4 | Aggregators & blogs | Hacker News, The Verge, Yahoo Finance |

### Propaganda Risk Assessment

Sources are tagged with risk levels:
- **High**: State-controlled (Xinhua, TASS, RT, CGTN, Press TV, KCNA)
- **Medium**: State-affiliated but editorially independent (Al Jazeera, France 24, DW News, VOA)
- **Low**: Independent with editorial standards (Reuters, AP, BBC, Guardian, Bellingcat)
