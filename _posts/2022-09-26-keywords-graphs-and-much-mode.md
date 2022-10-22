---
layout: post
title: "Weaponizing page rank for Fun and Profit, Part 1"
date: 2022-09-26 00:00:00 -0000
categories: [algorithms]
tags: [algorithm, graph, pagerank, keyword, search, ranking]
draft: true
---

### What is Page Rank?

Page Rank is a method for ranking web pages. It was created by Larry Page and Sergey Brin in 1998 at Stanford University. The original algorithm was based on the concept of a random surfer. It was later improved by Google to a more sophisticated algorithm. The current version of Page Rank is used by Google to rank web pages for their search engine. The Page Rank algorithm is a variant of eigenvector centrality. A page's rank is determined by the number of links pointing to it, as well as the rank of the pages linking to it. A page's rank can be calculated by the following formula:

$$
    PR(A) = \frac{1 - d}{N} + d \sum_{B \in In(A)} \frac{PR(B)}{Out(B)}
$$

where:

- $PR(A)$ is the page rank of page $A$.
- $d$ is the damping factor. It is the probability that a user will not click on a link. It is usually set to 0.85.
- $N$ is the number of pages in the graph.
- $In(A)$ is the set of pages that link to $A$.
- $Out(B)$ is the number of links from page $B$.

The formula might look a bit intimidating, but what it basically is trying to respresent is that, the more "good" websites have a link to your site, the more "good" your website is assumed to be. Essentially, if BBC news site links to your website, then it is as if BBC is vouching for the "goodness" of your website. The more links you have, the more "good" your website is assumed to be. The more "good" your website is assumed to be, the higher your page rank will be.

### How does Page Rank work

Page Rank works by crawling the web and collecting data about the links between websites. It then uses the data to calculate the page rank for each website. It then uses the page rank to rank the websites. The higher the page rank, the higher the rank of the website.

## Let create our own Page Rank graph

### Plan

To create our very own little Page Rank graph, we will need to do the following:

1. Create a web scraper to crawl the web and collect data about the links between websites.
   - We can break this down into two components:
     1. A web crawler that takes a website and returns its page content.
     2. A web parser that takes the page content and returns the links on the page.
   - Breaking this up into two components will allow us to reuse both components for other purposes. (which we will do later)
2. Create a graph from the data collected by the web scraper.
3. Calculate the page rank for each website.
4. Visualize the page rank data we have.

And to achieve this, we will be using:

- Node.js
  - for the web scraper, graph, and page rank calculation
- Vis.js
  - for the visualization

The internet is dark and full of terrors. If we set out to crawl even one millionth of the internet, we will be in for a long and painful journey. So, to make things easier, we will be crawling a small subset of the internet. We will be crawling from a specific set of website we define, to a certain depth and only following links that end with `.et`. This will allow us to crawl a small subset of the internet, focus mostly on local sites, and hopefully not get lost in the vastness of the internet.

## The Scraper

"Scraping? Isn't that just a fancy word for fetching data?"...

Yes, it is. But, it is also a fancy word for fetching data in a way that is not intended by the website. So, we can send out a request to the website, pretending to be a real user, and then parse the response to get the data we want.

Let's first start by installing the neccecary dependencies and structure our project.

```bash
mkdir page-rank-graph
cd page-rank-graph

npm init -y
npm install axios cheerio
```

Awesome! We now have a project with `axios` and `cheerio` installed. `axios` is a library for making HTTP requests. `cheerio` is a library for parsing HTML. We will be using them to make HTTP requests and parse the responses using jQuery-like syntax.

Now, let's create our entry point file `index.js` and start of with what we want our finished script to look like.

```js
const scraper = require("./scraper");

async function main() {
  const urlsToScrape = ["https://addissoftware.com"];

  const scrapedData = await scraper({
    urls: urlsToScrape,
    depth: 1,
    filter: (url) => {
      const urlObj = new URL(url);
      let isLocalSite = urlObj.hostname.endsWith(".et");
      let isInternalLink = urlsToScrape.filter((url) =>
        url.includes(urlObj.hostname)
      ).length;
      return isInternalLink || isLocalSite;
    },
    perPageLimit: 20,
  });

  console.log(JSON.stringify(scrapedData, null, 2));

  process.exit(0);
}

main();
```

{: file="index.js"}

We haven't yet implemented the `scraper` function, but we know what we want it to do. It should take a list of urls to scrape, a depth to scrape to, and a filter function. It should then return a list of scraped data. The scraped data should contain the url, the links on the page, and the depth of the page. We also want to limit the number of links we scrape per page. This is to prevent us from a huge number of links and a huge graph. If a page has links to 100 other pages, and each page also has links to 100 other pages, then we will end up with a graph with 10,000 nodes for depth 2, 1,000,000 nodes for depth 3, and so on. This will make our graph too big to visualize. So, we will limit the number of links we scrape per page.

Because we now have a clear idea of what we want our scraper to do, we can start implementing it. Let's continue with the `scraper.js` file.

```js
const axios = require("axios");
const cheerio = require("cheerio");

const scraper = async ({ urls, depth, filter, perPageLimit }) => {
  const scrapedData = [];

  for await (const url of urls) {
    const data = await scrapePage({ url, depth, filter, perPageLimit });
    scrapedData.push(...data);
  }

  return scrapedData;
};
```

{: file="scraper.js"}

We start by creating an empty array to store the scraped data. Then, we loop through the urls we want to scrape and call the `scrapePage` function for each url. The `scrapePage` function will return the scraped data for the page.

As you may have noticed, we are creating a lot of functions that are not yet implemented. This can help us in a multitude of ways when we are developing our code. First is that it helps us to focus on a single task at a time. Second is that it helps us to keep our code organized, free of additional effort. Third is that it helps us to keep track of what we have done and what we still need to do. And lastly, it helps build small, atomic, and reusable functions that can be used in other projects. We will see more about this later.

Let's continue with the `scrapePage` function.

```js
const scrapePage = async ({ url, depth, filter, perPageLimit }) => {
  if (depth === 0) return [];

  const pageContent = await getPageContent(url);
  const currentPageLinks = parsePageLinks(pageContent, {
    filter,
    url,
    perPageLimit,
  });

  const childPageLinks = [];
  for (const link of currentPageLinks) {
    const data = await scrapePage({
      url: link,
      depth: depth - 1,
      filter,
      perPageLimit,
    });
    childPageLinks.concat(data);
  }

  return [
    {
      url,
      links: currentPageLinks,
      depth,
    },
    ...childPageLinks,
  ];
};
```

{: file="scraper.js"}

This function has a bit more going on.
First, we check if the depth is 0. If it is, we return an empty array. This is because we don't want to scrape any further. So we return an empty array to indicate that we have no data to return.

Then we get the contents of the url using a new `getPageContent` function. We then parse the links on the page using a new `parsePageLinks` function. We also provide the `filter` function and the `url` to the `parsePageLinks` function. This is because we want to filter the links we get from the page and we want to use the url to create absolute links from relative links.

At this point, we have the links on the current page. We now need to follow those links and scrape the pages they point to. To do this, we loop through the links and call the same `scraperPage` function for each link, recursively.

Finally, we aggregate the data from the current page and the data from the child pages and return it.

We now only need two small functions to finish our scraper. Let's start with the `getPageContent` function.

```js
const getPageContent = async (url) => {
  const response = await axios.get(url);
  return response.data;
};
```

{: file="scraper.js"}

This function simply makes a GET request to the url and returns the response data.

Finally, let's implement the `parsePageLinks` function.

```js
const parsePageLinks = (pageContent, { filter, url, perPageLimit }) => {
  const $ = cheerio.load(pageContent);
  const links = $("a")
    .map((_, el) => $(el).attr("href"))
    .get();
  const sanitizedLinks = sanitizeLinks(links, url);
  if (filter) {
    return sanitizedLinks.filter(filter);
  }
  return sanitizedLinks.slice(0, perPageLimit);
};
```

{: file="scraper.js"}

This function uses `cheerio` to parse the page content. To break it down simply it:

- `$("a")` selects all the `a` tags in the page content.
- `map` loops through the `a` tags
  - `$(el).attr("href")` gets the `href` attribute of the `a` tag
- `get` executes the `map` function and returns the result as an array

We then sanitize the links using a new `sanitizeLinks` function. This is because some links are relative and some are absolute. For example, the link `/about` is relative to the current url, while the link `https://www.addissoftware.com/about` is absolute. We want to convert all the links to absolute links.

Finally, we filter and return the sanitized links. We also slice the links to the number of links we want to scrape per page. For now, we are just taking the first 20 links. We could implement some logic to take the most important links, but we will leave that for another time.

Let's implement the `sanitizeLinks` function.

```js
const sanitizeLinks = (links, url) => {
  const urlObj = new URL(url);

  const sanitizedLinks = [];
  for (const link of links) {
    if (link.startsWith("http")) {
      sanitizedLinks.push(link);
    } else {
      const sanitizedLink = `${urlObj.origin}${link}`;
      sanitizedLinks.push(sanitizedLink);
    }
  }
  return sanitizedLinks;
};
```

{: file="scraper.js"}

The `URL` node class gives us a lot of useful methods to work with urls. We use it to get the origin of the url. In the case of `"https://www.addissoftware.com/about"`, the origin is `"https://www.addissoftware.com"`, while the hostname is `"www.addissoftware.com"`.
Then we loop through the links and check if they start with `http`. If they do, it means they are absolute links and we can add them to the `sanitizedLinks` array. Otherwise, we create an absolute link by concatenating the origin of the url with the relative link.

We now have a fully functional scraper. Let's test it out.

```js
const scraper = require("./scraper");

async function main() {
  const urlsToScrape = ["https://addissoftware.com"];

  const scrapedData = await scraper({
    urls: urlsToScrape,
    depth: 1,
    filter: (url) => {
      const urlObj = new URL(url);
      let isLocalSite = urlObj.hostname.endsWith(".et");
      let isInternalLink = urlsToScrape.filter((url) =>
        url.includes(urlObj.hostname)
      ).length;
      return isInternalLink || isLocalSite;
    },
  });

  console.log(JSON.stringify(scrapedData, null, 2));

  process.exit(0);
}

main();
```

Running the above gives us this beautiful looking output.

```json
[
  {
    "url": "https://addissoftware.com",
    "links": [
      "https://addissoftware.com",
      "https://addissoftware.com/portfolios/",
      "https://addissoftware.com/about/",
      "https://addissoftware.com/careers/",
      "https://addissoftware.com/blog/",
      "https://addissoftware.com/contact-us/",
      "https://addissoftware.com/portfolios/",
      "https://addissoftware.com/about/",
      "https://addissoftware.com/careers/",
      "https://addissoftware.com/blog/",
      "https://addissoftware.com/contact-us/",
      "https://addissoftware.com/contact-us",
      "https://addissoftware.com/contact-us",
      "https://addissoftware.com/contact-us/",
      "https://addissoftware.com/projects/link-builders/",
      "https://addissoftware.com/projects/banks-ethiopia/",
      "https://addissoftware.com/portfolios",
      "https://addissoftware.com/contact-us/",
      "https://addissoftware.com",
      "https://addissoftware.com/contact-us",
      "https://addissoftware.com/a-deeper-look-into-react-native-and-its-architecture-part-ii/",
      "https://addissoftware.com/a-deeper-look-into-react-native-and-its-architecture-part-i/",
      "https://addissoftware.com/2021/06/production-grade-docker-compose/",
      "https://addissoftware.com/blog",
      "https://addissoftware.com",
      "https://addissoftware.com/",
      "https://addissoftware.com/about/",
      "https://addissoftware.com/careers/",
      "https://addissoftware.com/contact-us/"
    ],
    "depth": 1
  }
]
```

As you can see, the scraper is working as expected. While it is not perfect, one major issue we will soon encounter is that it may get into an infinite loop if there are circular links. As an example, lets assume that the `https://addissoftware.com` page has a link to `https://addissoftware.com/about`. Then, the `https://addissoftware.com/about` page also has a link to `https://addissoftware.com`. This will cause the scraper to get into an infinite loop, or atleast a very long loop of useless requests. We can have come dynamic programming to the rescue. More specifically, memoization. We will implement this in the next section.

```js
const memoize = (fn) => {
  const cache = {};
  return (...args) => {
    const key = JSON.stringify(args);
    if (cache[key]) {
      return cache[key];
    }
    const result = fn(...args);
    cache[key] = result;
    return result;
  };
};
```

{: file="scraper.js"}

We have just created a higher order function that takes a function as an argument and returns a memoized version of that function. The details of how this works is beyond the scope of this article. If you are interested in learning more about memoization, let us know and we will write a separate article about it.

```js
// we can now wrap the `scrapePage` function with the `memoize` function
const scrapePage = memoize(async ({ url, depth, filter }) => {
  //...
});
```

{: file="scraper.js"}

## The Graph

In this part, we will implement the graph data structure. We will use the graph to store the scraped data. I have run the scraper on a list of urls and saved the scraped data in a file called `scrapedData.json`. The json file has the same format as the output of the scraper. We will use the data to create the graph.

```js
const fs = require("fs");
const Graph = require("./graph");
const scrapedData = require("./scrapedData.json");

const graph = new Graph();

// because we know the scraped data is an array of objects
for (const data of scrapedData) {
  const { url, links, depth } = data;
  graph.addNode(url);
  for (const link of links) {
    graph.addNode(link);
    graph.addEdge(url, link);
  }
}

const html = graph.toHTML();
fs.writeFileSync("graph.html", html);
```

{: file="generateGraph.js"}

The above code is pretty self explanatory. We loop through the scraped data and for each url, we add the url as a node to the graph. Then we loop through the links of that url and add them as nodes to the graph as well. Finally, we add an edge between the url and the links. This way, we have a graph with nodes representing urls and edges representing links between the urls.

We have laid out what the Graph class should look like and what it should do. Let's implement it.

```js
class Graph {
  constructor() {
    this.nodes = [];
    this.edges = [];
  }

  addNode(node) {
    if (!this.nodes.find((n) => n.id === node)) {
      this.nodes.push({ id: node, label: node });
    }
  }

  addEdge(from, to) {
    const fromNode = this.nodes.find((n) => n.id === from);
    const toNode = this.nodes.find((n) => n.id === to);
    this.edges.push({ from: fromNode.id, to: toNode.id });
  }

  toHTML() {
    return `
      <html>
        <head>
          <title>Graph</title>
          <style type="text/css">
            #mynetwork {
              width: 100%;
              height: 100%;
              border: 1px solid lightgray;
            }
          </style>
          <script type="text/javascript" src="https://unpkg.com/vis-network/standalone/umd/vis-network.min.js"></script>
        </head>
        <body>
          <div id="mynetwork"></div>
          <script type="text/javascript">
            var nodes = new vis.DataSet(${JSON.stringify(this.nodes)});
            var edges = new vis.DataSet(${JSON.stringify(this.edges)});
            var container = document.getElementById("mynetwork");
            var data = {
              nodes: nodes,
              edges: edges,
            };
            var options = {
              physics: {
                solver: "forceAtlas2Based",
              }
            };
            var network = new vis.Network(container, data, options);
          </script>
        </body>
      </html>
    `;
  }
}

module.exports = Graph;
```

{: file="graph.js"}

Fantastic! We have a graph data structure that can be used to represent the scraped data. We can now use the graph to visualize the scraped data. We have (statically) used the `vis.js` library to visualize the graph. The `toHTML` method of the graph class returns an html string, which we can write to a file. We can then open the file in the browser to see the graph.

<iframe src="http://path/to/your/graph.html" width="100%" height="500px"></iframe>

The overall graph looked like this image the first time I ran the scraper. The graph is interactive and you can zoom in and out of the graph. You can also drag the nodes around to see the links between the nodes.

## Conclusion

This has turned out to be a very long blog post in its own right. We have covered a lot of ground in this article. We have seen how to scrape a website and create a graph from the scraped data. We have also indirectly touched on how to structure small projects, how to plan them out, how to use small improvements to make big changes, and so on.

In the next article, we will see how to read what different graphs we generate mean, and how to use that knowledge to understand how Search engines ( and at the same time SEO wink wink ) work. Until then, happy scraping!
