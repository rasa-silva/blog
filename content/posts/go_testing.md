---
title: "Data and behavior"
date: 2018-01-23T18:34:51Z
tags: ["go", "java"]
---

On my Go learning journey, after looking at 'Dependency Management', I started looking into testing...

I've decided that doing a basic GitHub client to fetch updates from my favorite repos was a nice little experiment to start with...

That said, part of the first run at implementing this was something like 

```go

func fetchCommits(repoName string) ([]commit, error) {
	url := fmt.Sprintf("https://api.github.com/repos/%s/commits", repoName)

	body, err := fetch(url)
	if err != nil {
		return nil, err
	}

	var commits []commit
	if err := json.Unmarshal(body, &commits); err != nil {
		return nil, err
	}

	return commits, nil
}

```

The fact that the URL is hardcoded is not a problem _per se_ (I don't think Github will change the endpoint or the API any time soon).

The problem is that it's not testable. 

If we try to test `fetchCommit()` we'll hit the real API endpoints no matter what...not good...

In a class based OO world, the code above could be something like

```java

class GitHubAPI {

    private final String baseURL;

    GitHubAPI(String baseURL) {
        this.baseURL = baseURL;
    }

    public fetchCommits(String repo) {
        ...
    }

}
```

But how to do this in Go?...

A class has two parts: data and behavior.

In Java, the data part is modeled as _class fields_. The behavior is modeled as _class methods_. If a class is just data, we usually call it a DTO or a Holder. 

In Go, data is modeled by basic types such as `int`, `bool`, or `string`. It can also be a collection of types, called `struct`.

Behavior is simply defined by functions. We can also _bind_ behavior to a type with a function (called a method in this case), via a special parameter called a _receiver_.

Let's do that

```go
type githubAPI struct {
	BaseUrl string

}

func (api githubAPI) fetchCommits(repoName string) ([]commit, error) {
	url := fmt.Sprintf("%s/repos/%s/commits", api.BaseURL, repoName)

	...
}

```

This is the same solution presented in the Java code but with its parts decomposed.

Now it can be tested

```go
func TestFetchCommits(t *testing.T) {

	ts := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		w.WriteHeader(http.StatusOK)
		w.Header().Set("Content-Type", "application/json")
		io.WriteString(w, `[ <json payload>	]`)
	}))

	defer ts.Close()

	api := githubAPI{BaseURL: ts.URL}

	commits, err := api.fetchCommits("someRepo")

	if err != nil {
		t.Fatal(err)
	}

	if len(commits) != 1 {
		t.Fatal("Commits len should be 1")
	}

}
```