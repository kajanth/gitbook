# cheatsheet

## Install

### Manually

Download it from https://golang.org/dl/

Extract it.

```text
tar -xzvf go1.14.4.linux-amd64.tar.gz
sudo mv go /usr/local/
```

Export env vars.

{% code title="vim ~/.zshrc" %}
```text
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```
{% endcode %}

### Ubuntu \(apt\)

```text
sudo add-apt-repository ppa:longsleep/golang-backports
sudo apt-get update
sudo apt-get install golang-go
```

Add to your `~/.bashrc`

```text
export GOPATH=$HOME/go
export PATH=$PATH:/usr/local/go/bin:$GOPATH/bin
```

## Libs / Frameworks

### Cobra

{% embed url="https://github.com/spf13/cobra" %}

## Async

```text
package main

import (
	"fmt"
	"net/http"
)

func main() {
	// A slice of sample websites
	urls := []string{
		"https://www.easyjet.com/",
		"https://www.skyscanner.de/",
		"https://www.ryanair.com",
		"https://wizzair.com/",
		"https://www.swiss.com/",
	}

	c := make(chan urlStatus)
	for _, url := range urls {
		go checkUrl(url, c)

	}
	result := make([]urlStatus, len(urls))
	for i, _ := range result {
		result[i] = <-c
		if result[i].status {
			fmt.Println(result[i].url, "is up.")
		} else {
			fmt.Println(result[i].url, "is down !!")
		}
	}

}

//checks and prints a message if a website is up or down
func checkUrl(url string, c chan urlStatus) {
	_, err := http.Get(url)
	if err != nil {
		// The website is down
		c <- urlStatus{url, false}
	} else {
		// The website is up
		c <- urlStatus{url, true}
	}
}

type urlStatus struct {
	url    string
	status bool
}
```

### References

[https://medium.com/@gauravsingharoy/asynchronous-programming-with-go-546b96cd50c1](https://medium.com/@gauravsingharoy/asynchronous-programming-with-go-546b96cd50c1)

