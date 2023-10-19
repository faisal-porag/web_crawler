# web crawler
Simple web crawler 


```go

package main

import (
	"fmt"
	"golang.org/x/net/html"
	"net/http"
	"strings"
	"sync"
)

// Crawler struct represents the web crawler.
type Crawler struct {
	Visited map[string]bool
	Mutex   sync.Mutex
}

// NewCrawler initializes a new crawler.
func NewCrawler() *Crawler {
	return &Crawler{
		Visited: make(map[string]bool),
	}
}

// VisitPage crawls a given URL and extracts links.
func (c *Crawler) VisitPage(url string, wg *sync.WaitGroup, sem chan struct{}) {
	defer wg.Done()

	c.Mutex.Lock()
	defer c.Mutex.Unlock()

	if c.Visited[url] {
		return
	}

	fmt.Println("Crawling:", url)
	c.Visited[url] = true

	resp, err := http.Get(url)
	if err != nil {
		fmt.Println("Error fetching", url, ":", err)
		return
	}
	defer resp.Body.Close()

	tokenizer := html.NewTokenizer(resp.Body)
	for {
		tokenType := tokenizer.Next()
		switch tokenType {
		case html.ErrorToken:
			return
		case html.StartTagToken, html.SelfClosingTagToken:
			token := tokenizer.Token()
			if token.Data == "a" {
				for _, attr := range token.Attr {
					if attr.Key == "href" {
						link := attr.Val
						if strings.HasPrefix(link, "http") {
							fmt.Println("Found link:", link)
							wg.Add(1)
							go c.VisitPage(link, wg, sem)
						}
					}
				}
			}
		}
	}
}

func main() {
	seedURL := "https://example.com" // Replace with the URL you want to start crawling from
	maxConcurrentRequests := 5

	crawler := NewCrawler()
	var wg sync.WaitGroup
	sem := make(chan struct{}, maxConcurrentRequests)

	// Start crawling from the seed URL.
	wg.Add(1)
	go crawler.VisitPage(seedURL, &wg, sem)

	// Wait for all goroutines to finish.
	wg.Wait()
}


```

> Run command

```sh
go run main.go
```



