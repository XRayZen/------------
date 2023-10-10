# GoQuery
```go
func getHtmlGoQueryDoc(url string) (*goquery.Document, error) {
	// /を消す
	if url[len(url)-1] == '/' {
		url = url[:len(url)-1]
	}
	resp, err := http.Get(url)
	if err != nil {
		return nil, fmt.Errorf("HTTP GET error: %v", err)
	}
	defer resp.Body.Close()
	if resp.StatusCode != http.StatusOK {
		return nil, fmt.Errorf("HTTP status error: %d", resp.StatusCode)
	}
	doc, err := goquery.NewDocumentFromReader(resp.Body)
	if err != nil {
		return nil, fmt.Errorf("goquery error: %v", err)
	}
	return doc, nil
}

func getHtmlGoQueryDoc(url string) (*goquery.Document, error) {
	// /を消す
	if url[len(url)-1] == '/' {
		url = url[:len(url)-1]
	}
	resp, err := http.Get(url)
	if err != nil {
		return nil, fmt.Errorf("HTTP GET error: %v", err)
	}
	defer resp.Body.Close()
	if resp.StatusCode != http.StatusOK {
		return nil, fmt.Errorf("HTTP status error: %d", resp.StatusCode)
	}
	doc, err := goquery.NewDocumentFromReader(resp.Body)
	if err != nil {
		return nil, fmt.Errorf("goquery error: %v", err)
	}
	return doc, nil
}
```














































