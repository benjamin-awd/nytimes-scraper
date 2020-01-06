# nyt-scraper
Scrape article metadata comments from NYTimes

## Setup
1. Install the [Anaconda python runtime](https://anaconda.org/)
1. Create the python environment with
   `conda env create -f environment.yml`
   or update an existing `nyt-scraper` environment to match remote dependency changes by executing
   `conda env update -f environment.yml --prune`
1. Activate the conda environment
   `conda activate nyt-scraper`

## CLI
The scraper will automatically fetch metadata and comments for every article published on
[nytimes.com].
Articles are processed month by month, starting with the current month.
For each month, a `{year}-{month}-article.pickle` and `{year}-{month}-comments.pickle` will be
generated in the `data` directory.

1. Create a copy of `.env.example` in the repository root directory and name it `.env`
1. Update the values in `.env` as needed
1. Run the service
   `python main.py`

## Programmatically
The scraper can also be started programmatically
```python
import datetime as dt
from nyt_scraper import run_scraper, scrape_month

# scrape february of 2020
article_df, comment_df = scrape_month('<your_api_key>', date=dt.date(2020, 2, 1))

# scrape all articles, similar to CLI
run_scraper('<your_api_key>')
```

Alternatively, the `nyt_scraper.articles` and `nyt_scraper.comments` modules can be used for more
fine-grained control:
```python
import datetime as dt
from nyt_scraper.nyt_api import NytApi
from nyt_scraper.articles import fetch_articles_by_month, articles_to_df
from nyt_scraper.comments import fetch_comments, fetch_comments_by_article, comments_to_df

api = NytApi('<your_api_key>')

# Fetch articles of a specific month
articles = fetch_articles_by_month(api, dt.date(2020, 2, 1))
article_df = articles_to_df(articles)

# Fetch comments from multiple articles
# a) using the results of a previous article query
article_ids_and_urls = list(article_df['web_url'].iteritems())
comments_a = fetch_comments(api, article_ids_and_urls)
comment_df = comments_to_df(comments_a)

# b) using a custom list of articles
comments_b = fetch_comments(api, article_ids_and_urls=[
    ('nyt://article/316ef65c-7021-5755-885c-a9e1ef2cfdf2', 'https://www.nytimes.com/2020/01/03/world/middleeast/trump-iran-suleimani.html'),
    ('nyt://article/b2d1b802-412e-51f7-8864-efc931e87bb3', 'https://www.nytimes.com/2020/01/04/opinion/impeachment-witnesses.html'),
])

# Fetch comment for one specific article by its URL
comments_c = fetch_comments_by_article(api, 'https://www.nytimes.com/2019/11/30/opinion/sunday/bernie-sanders.html')
```
