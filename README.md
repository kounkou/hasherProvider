[![Go](https://github.com/kounkou/hasherProvider/workflows/Go/badge.svg)](https://github.com/kounkou/hasherProvider/actions?query=workflow%3AGo)
[![Coverage Status](https://coveralls.io/repos/github/kounkou/hasherProvider/badge.svg?branch=main)](https://coveralls.io/github/kounkou/hasherProvider?branch=main)
[![Go Report Card](https://goreportcard.com/badge/github.com/kounkou/hasherProvider)](https://goreportcard.com/report/github.com/kounkou/hasherProvider)
[![gopkg](https://pkg.go.dev/badge/github.com/kounkou/hasherProvider.svg)](https://pkg.go.dev/github.com/kounkou/hasherProvider)
[![license](https://img.shields.io/badge/License-MIT-blue.svg)](https://github.com/josuebrunel/clausify/blob/master/LICENSE)

# hasherProvider

The Hasher library implements a hashing algorithm on a given key string or UUID and returns the server to which the request should be sent. 
Consistent hashing is one such algorithm that minimizes the number of updates required to associate the request with the appropriate server. 
This addresses the common problem of reassigning servers that arises when using the modulo operation.

# Installation

```bash
# Step 1 : Create test project directory
mkdir test && cd test

# Step 2 : Create test.go file and copy code snippet into test.go
touch test.go

# Step 3 : Fetch dependencies
go mod init test && go mod tidy

# Step 5 : launch test, and observe success !
go run test.go
```

# Usage

Example : 
3 servers are hashed based on their id + replica number. 
Here is the structure of the `ConsistentHashing` object.

```
1205995440:server2 --> 1306808249:server3 --> 3655969099:server1
```

In this example, we hash `9` which has hash 

```
1007465396
```

Let's see the representation of the both output on an hypothetical graph, and let's insert the `9` inside the graph : 

```
1007465396:9 --> 1205995440:server2 --> 1306808249:server3 --> 3655969099:server1
```

This means that the closest node clockwise for the `9` is server2, which is expected result for below code.

```golang
package main

import (
    "fmt"
    "github.com/kounkou/hasherProvider"
)

const (
	CONSISTENT_HASHING = 0
	RANDOM_HASHING     = 1
	UNIFORM_HASHING    = 2
)

func main() {
	// Create a new HasherProvider object
	hasherProvider := hasherProvider.HasherProvider{ }

	// Get the consistent hashing function
	h, err := hasherProvider.GetHasher(CONSISTENT_HASHING)

    	// Set replicas entities
	h.SetReplicas(1)

	if h == nil || err != nil {
		fmt.Println("Error getting hasher:", err)
		return
	}

    	h.AddNode("server1")
	h.AddNode("server2")
    	h.AddNode("server3")

    	result, err := h.Hash("9", 0)

    	if err != nil {
        	fmt.Println("Error getting hash for some string `9` ", err)
    	}

    	if result != "server2" {
        	fmt.Errorf("Expected replica to be assigned is server2")
        	return
    	}

    	fmt.Println("Success... !")
}
```

# Algorithms

HasherProvider currently supports 3 algorithms. You might want to choose your hashing algorithm based on the following characteristics :

| Hashing Algorithm  | Load balanced | Elastic   | Fault tolerant | Decentralized |
|--------------------|---------------|-----------|----------------|---------------|
| Consistent Hashing | Excellent     | Excellent | Excellent      | Excellent     |
| Random Hashing     | Good          | Poor      | Good           | Poor          |
| Uniform Hashing    | Poor          | Good      | Poor           | Excellent     |

