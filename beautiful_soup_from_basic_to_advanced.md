# Beautiful Soup (bs4): From Beginner to Advanced
## A practical Python web-scraping reference

**Goal:** learn to parse HTML/XML, find elements, extract data, navigate a document, handle messy HTML, and build reliable scraping scripts.

> Beautiful Soup parses HTML/XML that you give it. It does not download web pages by itself. Use `requests` (or Python's `urllib`) to fetch a page, then Beautiful Soup to parse its response.

## 1. What is Beautiful Soup?

Beautiful Soup is a Python library for reading and searching HTML and XML documents.

Common uses:
- Extract headings, paragraphs, links, image URLs, tables, and lists.
- Parse HTML saved in a file.
- Parse HTML returned by an HTTP request.
- Process XML documents.
- Clean or modify markup in memory.

Beautiful Soup is not a browser, does not execute JavaScript, and does not bypass logins, CAPTCHAs, paywalls, or access controls. If a site generates content with JavaScript, the initial HTML response may not contain the data you see in the browser. Look for an authorized API or documented data source.

## 2. Installation

### Windows

```powershell
py -m pip install beautifulsoup4
py -m pip install lxml
```

### Linux / WSL

```bash
python3 -m pip install beautifulsoup4
python3 -m pip install lxml
```

If your system Python is externally managed, use a virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install beautifulsoup4 lxml
```

Windows virtual environment:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install beautifulsoup4 lxml
```

Verify:

```bash
python -c "from bs4 import BeautifulSoup; print('Beautiful Soup is ready')"
```

| Item | Name |
|---|---|
| Package installed with pip | `beautifulsoup4` |
| Python import module | `bs4` |
| Main class | `BeautifulSoup` |
| Built-in parser | `html.parser` |
| Optional parser | `lxml` |
| Optional HTML5 parser | `html5lib` |

Install `html5lib` only if you need it:

```bash
python -m pip install html5lib
```

## 3. Your first example

Create `first_soup.py`:

```python
from bs4 import BeautifulSoup

html = """
<html>
  <head><title>My Learning Page</title></head>
  <body>
    <h1>Learn Python</h1>
    <p class="intro">Beautiful Soup makes HTML easier to search.</p>
  </body>
</html>
"""

soup = BeautifulSoup(html, "html.parser")

print(soup.title)                 # Complete <title> tag
print(soup.title.get_text())      # Text inside the title
print(soup.h1.get_text())         # Text inside the first h1
```

Run it:

```bash
python first_soup.py
```

In `BeautifulSoup(html, "html.parser")`, `html` is the source text, `"html.parser"` is the parser, and `soup` is the parsed document tree.

## 4. Tags, attributes, and text

Given:

```html
<p class="notice" id="message">Hello <strong>Python</strong> learner!</p>
```

- Tag name: `p`
- Attributes: `class`, `id`
- Attribute values: `notice`, `message`
- Nested tag: `strong`
- Text: `Hello Python learner!`

```python
from bs4 import BeautifulSoup

html = '<p class="notice" id="message">Hello <strong>Python</strong> learner!</p>'
soup = BeautifulSoup(html, "html.parser")
p = soup.find("p")

print(p.name)                 # p
print(p.get("id"))            # message
print(p.get("class"))         # ['notice']; class is multi-valued
print(p.get_text())           # Hello Python learner!
print(p.strong.get_text())    # Python
print(p["id"])                # message; KeyError if absent
print(p.get("missing"))       # None if absent
```

Prefer `tag.get("attribute")` when an attribute might be missing. Direct indexing such as `tag["href"]` raises `KeyError` if the attribute does not exist.

## 5. Choosing a parser

```python
BeautifulSoup(html, "html.parser")
BeautifulSoup(html, "lxml")
BeautifulSoup(html, "html5lib")
```

- `html.parser`: built into Python; a good starting point.
- `lxml`: third-party parser, generally fast and useful for many HTML/XML tasks.
- `html5lib`: aims to parse HTML similarly to a modern browser, but may be slower.

Parser choice can affect how malformed HTML is repaired. Specify the parser explicitly so your script behaves predictably.

## 6. Find one element with `find()`

`find()` returns the first match, or `None` if there is no match.

```python
html = """
<h1>News</h1>
<p class="story">First story</p>
<p class="story">Second story</p>
"""
soup = BeautifulSoup(html, "html.parser")

print(soup.find("h1").get_text())
print(soup.find("p").get_text())  # First p
print(soup.find("p", class_="story").get_text())
print(soup.find("h2"))            # None
```

Handle a missing element safely:

```python
heading = soup.find("h2")

if heading is not None:
    print(heading.get_text(strip=True))
else:
    print("No h2 heading found")
```

## 7. Find multiple elements with `find_all()`

`find_all()` returns a list-like `ResultSet`.

```python
html = """
<ul>
  <li>Apples</li>
  <li>Bananas</li>
  <li>Oranges</li>
</ul>
"""
soup = BeautifulSoup(html, "html.parser")

items = soup.find_all("li")
print(len(items))  # 3

for item in items:
    print(item.get_text(strip=True))
```

Common patterns:

```python
soup.find_all("a")                    # All links
soup.find_all("img")                  # All images
soup.find_all("p")                    # All paragraphs
soup.find_all(["h1", "h2", "h3"])     # Any listed tag name
soup.find_all(class_="product")       # Elements with class product
soup.find_all(id="main")              # Element(s) with id main
soup.find_all("li", limit=2)          # At most two matches
```

## 8. CSS selectors: `select()` and `select_one()`

CSS selectors are a concise way to locate elements.

```python
html = """
<div id="catalog">
  <article class="product featured">
    <h2 class="name">Keyboard</h2>
    <span class="price">$25</span>
    <a href="/keyboard">Details</a>
  </article>
  <article class="product">
    <h2 class="name">Mouse</h2>
    <span class="price">$12</span>
  </article>
</div>
"""
soup = BeautifulSoup(html, "html.parser")

print(soup.select_one("#catalog"))        # First id=catalog
print(soup.select(".product"))            # All product classes
print(soup.select(".product.featured"))   # Has both classes
print(soup.select(".product .name"))      # name anywhere inside product
print(soup.select("article > .price"))    # Direct child price
print(soup.select("a[href]"))             # Links with href
print(soup.select('a[href^="/"]'))        # href starts with /
print(soup.select('a[href$="keyboard"]')) # href ends with keyboard
print(soup.select('a[href*="key"]'))      # href contains key
```

| Selector | Meaning | Example |
|---|---|---|
| `p` | Every `p` tag | `select("p")` |
| `.price` | Class `price` | `select(".price")` |
| `#main` | ID `main` | `select("#main")` |
| `div p` | A `p` anywhere inside a `div` | `select("div p")` |
| `div > p` | Direct child `p` | `select("div > p")` |
| `[href]` | Has `href` | `select("a[href]")` |
| `[href^="https"]` | Starts with `https` | `select('[href^="https"]')` |
| `[href$=".pdf"]` | Ends with `.pdf` | `select('[href$=".pdf"]')` |
| `[href*="docs"]` | Contains `docs` | `select('[href*="docs"]')` |

`select_one()` returns the first match or `None`. `select()` returns all matches. Beautiful Soup supports many common CSS selectors, but not every feature of a full browser engine.

## 9. Extract text correctly

```python
html = "<p>Learn <b>Python</b> today</p>"
soup = BeautifulSoup(html, "html.parser")
p = soup.p

print(p.get_text())                   # Learn Python today
print(p.get_text(" ", strip=True))    # Learn Python today
print(list(p.stripped_strings))       # ['Learn', 'Python', 'today']
```

`get_text(separator="", strip=False)`:
- `separator`: text inserted between text pieces.
- `strip`: trims whitespace around text pieces.

For nested markup or inconsistent whitespace, `get_text(" ", strip=True)` is often useful.

## 10. Read attributes: links, images, data attributes

```python
html = """
<a href="https://example.com/docs" target="_blank">Documentation</a>
<img src="/images/flower.png" alt="A flower">
<div data-product-id="p123">Flower pot</div>
"""
soup = BeautifulSoup(html, "html.parser")

link = soup.find("a")
print(link.get_text(strip=True))
print(link.get("href"))
print(link.get("target"))

image = soup.find("img")
print(image.get("src"))
print(image.get("alt"))

product = soup.find("div", attrs={"data-product-id": True})
print(product.get("data-product-id"))
```

`tag.attrs` returns the attribute dictionary. Use `tag.get("href")` for safe lookups. For hyphenated attributes like `data-product-id`, use `.get("data-product-id")` or `attrs={...}` because hyphens cannot be used in normal Python keyword names.

HTML `class` attributes are usually represented as a list:

```python
classes = tag.get("class", [])
if "featured" in classes:
    print("Featured element")
```

## 11. Navigate the document tree

Example:

```html
<div id="container">
  <h2>Catalog</h2>
  <p>Intro</p>
  <ul>
    <li>Book</li>
    <li>Pen</li>
  </ul>
</div>
```

```python
container = soup.find(id="container")

container.parent
container.contents       # Direct children, including text nodes
container.children       # Iterator over direct children
container.descendants    # Iterator over all descendants
container.find_all("li")
container.find("ul")
```

Parent and sibling navigation:

```python
item = soup.find("li")

item.parent
item.find_parent("div")
item.find_next_sibling("li")
item.find_previous_sibling("li")
item.find_next("p")
item.find_previous("h2")
```

Notes:
- `.parent` is a property, not a method.
- `.children` means direct children; `.descendants` includes nested descendants.
- `find_next()` and `find_previous()` search in document order, not only siblings.
- `next_sibling` can be a whitespace text node. `find_next_sibling("li")` searches for a matching tag.

## 12. Search by multiple conditions, functions, and regular expressions

```python
soup.find_all("a", href=True)
soup.find_all("a", class_="external")
soup.find_all("div", id="main")
soup.find_all(["h1", "h2"])
soup.select("div.card a[href]")
```

Custom callable filter:

```python
def has_data_id(tag):
    return tag.has_attr("data-id")

matches = soup.find_all(has_data_id)
```

Regular expression filter:

```python
import re
from bs4 import BeautifulSoup

html = """
<a href="/products/101">One</a>
<a href="/products/202">Two</a>
<a href="/about">About</a>
"""
soup = BeautifulSoup(html, "html.parser")

product_links = soup.find_all("a", href=re.compile(r"^/products/\d+$"))

for link in product_links:
    print(link.get("href"), link.get_text(strip=True))
```

Use CSS selectors for ordinary structural matching; use regex when it genuinely simplifies matching. Keep custom filters small and account for missing or malformed attributes.

## 13. Fetch HTML using `requests`

Install requests:

```bash
python -m pip install requests
```

Example:

```python
import requests
from bs4 import BeautifulSoup

url = "https://example.com/"

try:
    response = requests.get(
        url,
        timeout=15,
        headers={"User-Agent": "LearningScraper/1.0"}
    )
    response.raise_for_status()

    soup = BeautifulSoup(response.text, "html.parser")
    title = soup.title.get_text(strip=True) if soup.title else "No title"
    print("Page title:", title)

    for link in soup.select("a[href]"):
        print(link.get_text(" ", strip=True), link.get("href"))

except requests.RequestException as error:
    print("Request failed:", error)
```

Workflow:
1. `requests.get()` sends an HTTP GET request.
2. `timeout=15` limits waiting time.
3. `raise_for_status()` raises an error for HTTP 4xx/5xx responses.
4. `response.text` is decoded response content.
5. Beautiful Soup parses that HTML.
6. Selectors locate the desired elements.

Use timeouts, handle errors, check that the expected page was returned, keep request rates reasonable, and respect site rules and access restrictions. Use an official API where appropriate. `requests` fetches; Beautiful Soup parses.

## 14. Read HTML from a local file

```python
from pathlib import Path
from bs4 import BeautifulSoup

html_path = Path("page.html")

with html_path.open("r", encoding="utf-8") as file:
    soup = BeautifulSoup(file, "html.parser")

print(soup.title.get_text(strip=True) if soup.title else "No title")
```

Use the actual file encoding if it is not UTF-8.

## 15. Parse HTML tables

```python
from bs4 import BeautifulSoup

html = """
<table>
  <thead><tr><th>Name</th><th>Price</th></tr></thead>
  <tbody>
    <tr><td>Keyboard</td><td>$25</td></tr>
    <tr><td>Mouse</td><td>$12</td></tr>
  </tbody>
</table>
"""
soup = BeautifulSoup(html, "html.parser")

for row in soup.select("tr"):
    cells = row.find_all(["th", "td"])
    values = [cell.get_text(" ", strip=True) for cell in cells]
    print(values)
```

Output:

```text
['Name', 'Price']
['Keyboard', '$25']
['Mouse', '$12']
```

For complex tables, consider nested tables, missing cells, `rowspan`, and `colspan`. The simple loop above does not automatically interpret row/column spans.

## 16. Extract product cards into dictionaries

```python
from bs4 import BeautifulSoup

html = """
<div class="product">
  <h2 class="name">Keyboard</h2>
  <span class="price">$25</span>
  <a class="details" href="/products/keyboard">View</a>
</div>
<div class="product">
  <h2 class="name">Mouse</h2>
  <span class="price">$12</span>
</div>
"""
soup = BeautifulSoup(html, "html.parser")
products = []

for card in soup.select(".product"):
    name_tag = card.select_one(".name")
    price_tag = card.select_one(".price")
    link_tag = card.select_one("a.details")

    products.append({
        "name": name_tag.get_text(" ", strip=True) if name_tag else None,
        "price": price_tag.get_text(" ", strip=True) if price_tag else None,
        "url": link_tag.get("href") if link_tag else None,
    })

print(products)
```

Checking for `None` avoids crashes when a field is missing. In a production scraper, validate required fields and log incomplete records.

## 17. Convert relative URLs to absolute URLs

```python
from urllib.parse import urljoin
from bs4 import BeautifulSoup

page_url = "https://example.com/catalog/"
html = '<a href="/products/1">Product</a><a href="details.html">Details</a>'
soup = BeautifulSoup(html, "html.parser")

for link in soup.select("a[href]"):
    print(urljoin(page_url, link["href"]))
```

`urljoin()` resolves URL references but does not verify that a URL exists or is safe. If automatically following links, validate scheme and allowed hostname:

```python
from urllib.parse import urljoin, urlparse

base_url = "https://example.com/catalog/"
candidate = urljoin(base_url, "/products/1")
parsed = urlparse(candidate)

if parsed.scheme == "https" and parsed.hostname == "example.com":
    print("Allowed URL:", candidate)
else:
    print("Unexpected URL:", candidate)
```

For larger systems, handle ports as needed and compare hostnames carefully. A loose string-suffix check can be unsafe.

## 18. Modify or remove markup

```python
from bs4 import BeautifulSoup

html = """
<div>
  <h1>Article</h1>
  <script>not article text</script>
  <style>.hidden { display: none; }</style>
  <p>Main article content.</p>
  <div class="advert">Advertisement</div>
</div>
"""
soup = BeautifulSoup(html, "html.parser")

for tag in soup.select("script, style, .advert"):
    tag.decompose()

print(soup.get_text(" ", strip=True))
```

Useful methods:
- `tag.decompose()`: remove the tag and contents, destroying that object in the tree.
- `tag.extract()`: remove the tag and return it.
- `tag.clear()`: remove children but keep the tag.
- `tag.unwrap()`: remove the tag but preserve its contents.
- `tag.string = "new text"`: replace string content.
- `tag.attrs`: inspect or change attributes.
- `soup.new_tag("p")`: create a tag.
- `tag.append(new_tag)`: add a child.
- `tag.insert(0, new_tag)`: insert at a position.
- `str(soup)`: serialize the modified tree.

Edits affect the in-memory tree, not the live website. To save modified HTML, write `str(soup)` to a file.

## 19. XML basics

Install `lxml` if needed:

```bash
python -m pip install lxml
```

```python
from bs4 import BeautifulSoup

xml = """
<library>
  <book id="b1">
    <title>Python Basics</title>
    <author>Alex</author>
  </book>
</library>
"""
soup = BeautifulSoup(xml, "xml")

book = soup.find("book")
print(book.get("id"))
print(book.title.get_text(strip=True))
print(book.author.get_text(strip=True))
```

XML is stricter than HTML. For namespace-heavy XML or complex XML querying, a dedicated XML library such as `lxml` may be more appropriate.

## 20. Advanced: `SoupStrainer`

`SoupStrainer` can restrict which parts of the input are parsed. This can help for large documents when you already know the elements you need.

```python
from bs4 import BeautifulSoup, SoupStrainer

html = """
<html>
  <head><title>Example</title></head>
  <body>
    <p>Intro</p>
    <div class="product">Keyboard</div>
    <div class="product">Mouse</div>
  </body>
</html>
"""
only_products = SoupStrainer("div", class_="product")
soup = BeautifulSoup(html, "html.parser", parse_only=only_products)

for product in soup.find_all("div", class_="product"):
    print(product.get_text(strip=True))
```

It is not always faster or necessary; profile before optimizing.

## 21. Advanced: custom filters

```python
from bs4 import BeautifulSoup

html = """
<div data-score="91">A</div>
<div data-score="45">B</div>
<div>No score</div>
"""
soup = BeautifulSoup(html, "html.parser")

def score_at_least_80(tag):
    if tag.name != "div":
        return False
    raw_score = tag.get("data-score")
    if raw_score is None:
        return False
    try:
        return int(raw_score) >= 80
    except ValueError:
        return False

for tag in soup.find_all(score_at_least_80):
    print(tag.get_text(strip=True), tag.get("data-score"))
```

A callable filter can express conditions that are awkward in a CSS selector. Handle missing and malformed values.

## 22. Advanced: reusable extraction function

```python
from bs4 import BeautifulSoup

def extract_page_summary(html: str) -> dict:
    """Extract common fields from an HTML document."""
    soup = BeautifulSoup(html, "html.parser")

    title_tag = soup.find("title")
    description_tag = soup.find("meta", attrs={"name": "description"})

    headings = [
        heading.get_text(" ", strip=True)
        for heading in soup.select("h1, h2, h3")
    ]

    links = [
        {
            "text": tag.get_text(" ", strip=True),
            "href": tag.get("href"),
        }
        for tag in soup.select("a[href]")
    ]

    return {
        "title": title_tag.get_text(" ", strip=True) if title_tag else None,
        "description": description_tag.get("content") if description_tag else None,
        "headings": headings,
        "links": links,
    }

sample_html = """
<html>
  <head>
    <title>Learning Page</title>
    <meta name="description" content="A sample learning page">
  </head>
  <body>
    <h1>Python</h1>
    <h2>Web scraping</h2>
    <a href="/guide">Guide</a>
  </body>
</html>
"""
print(extract_page_summary(sample_html))
```

For bigger projects, use more specific type annotations or a dataclass and add tests.

## 23. Save extracted data as JSON

```python
import json
from pathlib import Path

records = [
    {"name": "Keyboard", "price": "$25"},
    {"name": "Mouse", "price": "$12"},
]

output_path = Path("products.json")
output_path.write_text(
    json.dumps(records, indent=2, ensure_ascii=False),
    encoding="utf-8",
)
print(f"Saved {len(records)} records to {output_path}")
```

Separate extraction from storage: validate the Python data first, then serialize it.

## 24. Error handling and defensive extraction

Common problems include request timeouts, HTTP errors, missing tags, missing attributes, malformed numbers, and the site returning a consent or bot-check page instead of the expected content.

```python
from decimal import Decimal, InvalidOperation
from bs4 import BeautifulSoup

html = '<div class="price">$25.50</div>'
soup = BeautifulSoup(html, "html.parser")
price_tag = soup.select_one(".price")

if price_tag is None:
    print("Price element was not found")
else:
    raw_price = price_tag.get_text(" ", strip=True)
    cleaned_price = raw_price.replace("$", "").replace(",", "")
    try:
        price = Decimal(cleaned_price)
        print("Price:", price)
    except InvalidOperation:
        print("Price could not be parsed:", raw_price)
```

`Decimal` is often preferable to `float` for money. Currency formats vary, so this is only a simple example, not a universal price parser.

## 25. Common mistakes

### Wrong package/import
```bash
python -m pip install beautifulsoup4
```
```python
from bs4 import BeautifulSoup
```

### Calling `.text` on `None`
```python
heading = soup.find("h1")
print(heading.get_text(strip=True) if heading else "Heading missing")
```

### Expecting Beautiful Soup to fetch pages
Use `requests` or another HTTP client to fetch HTML.

### Expecting JavaScript to run
Beautiful Soup does not execute JavaScript. Inspect the HTTP response or use an authorized API if content is dynamic.

### Assuming `class` is a string
```python
classes = tag.get("class", [])
if "featured" in classes:
    print("Featured")
```

### Forgetting relative URLs
Use `urljoin()`.

### Ignoring status codes/timeouts
With requests, set `timeout=...` and call `raise_for_status()`.

### Relying on fragile page structure
Use clear selectors, handle missing fields, and test against saved HTML.

### Scraping too aggressively
Use reasonable request rates and follow applicable rules and access restrictions.

## 26. Complete small scraper (offline demo)

This example uses a local HTML string, so it runs without internet access.

```python
from urllib.parse import urljoin
from bs4 import BeautifulSoup

BASE_URL = "https://example.com/catalog/"

HTML = """
<html><body>
  <article class="product">
    <h2 class="name">Keyboard</h2>
    <span class="price">$25.00</span>
    <a href="/products/keyboard">Details</a>
  </article>
  <article class="product">
    <h2 class="name">Mouse</h2>
    <span class="price">$12.00</span>
    <a href="products/mouse">Details</a>
  </article>
  <article class="product">
    <span class="price">$8.00</span>
    <a href="/products/unknown">Details</a>
  </article>
</body></html>
"""

def scrape_products(html: str, base_url: str) -> list[dict[str, str | None]]:
    soup = BeautifulSoup(html, "html.parser")
    results = []

    for card in soup.select("article.product"):
        name_tag = card.select_one(".name")
        price_tag = card.select_one(".price")
        link_tag = card.select_one("a[href]")

        href = link_tag.get("href") if link_tag else None
        results.append({
            "name": name_tag.get_text(" ", strip=True) if name_tag else None,
            "price": price_tag.get_text(" ", strip=True) if price_tag else None,
            "url": urljoin(base_url, href) if href else None,
        })

    return results

if __name__ == "__main__":
    for product in scrape_products(HTML, BASE_URL):
        print(product)
```

The third product has no name; the code returns `None` rather than crashing. A production scraper might skip incomplete records or log a warning.

## 27. Test extraction logic without a live website

```python
from bs4 import BeautifulSoup

def extract_title(html: str) -> str | None:
    soup = BeautifulSoup(html, "html.parser")
    tag = soup.find("title")
    return tag.get_text(strip=True) if tag else None

def test_extract_title():
    assert extract_title("<title>Demo</title>") == "Demo"
    assert extract_title("<p>No title</p>") is None

test_extract_title()
print("Tests passed")
```

For larger applications, use `unittest` or `pytest`, include missing-element and malformed-data cases, and keep representative HTML fixtures. Saved fixtures make tests repeatable when the source website changes.

## 28. Performance and maintainability

1. Choose a parser explicitly.
2. Use readable CSS selectors for structural searches.
3. Use `select_one()` for one element and check for `None`.
4. Scope searches to a card or container instead of repeatedly searching the entire document inside loops.
5. Use `SoupStrainer` only if it helps a measured workload.
6. Separate fetching, parsing, validation, and storage into functions.
7. Set HTTP timeouts and handle expected errors.
8. Log useful errors in larger scripts.
9. Test with saved HTML.
10. Review selectors when the source page changes.

## 29. Ethical, legal, and security considerations

- Follow applicable laws and website terms.
- Respect authentication boundaries and access controls.
- Do not collect private or sensitive information without proper authorization.
- Prefer an official API when appropriate.
- Keep request rates modest.
- `robots.txt` is not, by itself, legal permission.
- Treat scraped text and URLs as untrusted input. Never execute downloaded scripts or evaluate scraped strings as Python code.
- If displaying scraped text in a web application, escape or safely render it to prevent injection.

## 30. Quick reference cheat sheet

### Create a soup
```python
soup = BeautifulSoup(html, "html.parser")
```

### Find elements
```python
soup.find("h1")
soup.find_all("p")
soup.select_one(".product")
soup.select("a[href]")
```

### Extract text
```python
tag.get_text()
tag.get_text(" ", strip=True)
list(tag.stripped_strings)
```

### Read attributes
```python
tag.get("href")
tag.get("src")
tag.get("class")
tag.attrs
```

### Navigate
```python
tag.parent
tag.children
tag.descendants
tag.find_parent("div")
tag.find_next_sibling("p")
```

### Modify markup
```python
tag.decompose()
tag.extract()
tag.unwrap()
tag.clear()
tag["class"] = ["active"]
new_tag = soup.new_tag("p")
new_tag.string = "Hello"
tag.append(new_tag)
```

### Fetch and parse
```python
import requests
from bs4 import BeautifulSoup

response = requests.get("https://example.com/", timeout=15)
response.raise_for_status()
soup = BeautifulSoup(response.text, "html.parser")
```

### Resolve relative URLs
```python
from urllib.parse import urljoin
absolute_url = urljoin("https://example.com/catalog/", "/products/1")
```

## 31. Practice roadmap

Try these exercises in order:

1. Parse a string with a title and heading.
2. Extract every paragraph with `find_all("p")`.
3. Extract all links and their `href` values.
4. Use CSS selectors to find `.product` cards.
5. Extract product names, prices, and links into dictionaries.
6. Handle missing headings and attributes.
7. Parse an HTML table into rows.
8. Resolve relative links with `urljoin()`.
9. Read an HTML file from disk.
10. Fetch a public page using requests, a timeout, and status checking.
11. Save extracted records as JSON.
12. Write tests using saved HTML.
13. Add logging for request errors and missing fields.
14. Use a custom filter for `data-*` attributes.
15. Compare `html.parser` and `lxml` on the same malformed HTML.

### Final mental model

A scraper usually has four stages:

1. **Fetch:** obtain HTML using `requests` or another HTTP client.
2. **Parse:** build the document tree with Beautiful Soup.
3. **Extract and validate:** locate tags, read text/attributes, and handle missing data.
4. **Store:** save validated data to JSON, CSV, a database, or another destination.

Beautiful Soup mainly handles stage 2 and helps with stage 3. Separating these stages makes a scraper easier to understand, debug, test, and maintain.

## Official documentation

- Beautiful Soup: https://www.crummy.com/software/BeautifulSoup/bs4/doc/
- Requests: https://requests.readthedocs.io/
- Python `urllib.parse`: https://docs.python.org/3/library/urllib.parse.html
- Python `json`: https://docs.python.org/3/library/json.html
