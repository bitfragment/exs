---
title: "Caching sequential web proxy"
date: 2018-11-29
tags: c, core
published: true
---

Source: [CS:APP3e, Bryant and O'Hallaron](http://csapp.cs.cmu.edu/), 
Proxy Lab (from [Lab Assignments](http://csapp.cs.cmu.edu/3e/labs.html)), 
using the downloadable self-study 
"[handout](http://csapp.cs.cmu.edu/3e/proxylab-handout.tar)."

The task is to implement a caching sequential web proxy.

The program here uses the Rio (Robust I/O) package in the provided library `csapp`.

It is based on the provided library `tiny` (a tiny web server).

* TOC
{:toc}

```c
#include <stdio.h>
#include "csapp.h"
```

## Cache

Cache is implemented as a doubly linked list. The head node represents the
freshest data, the tail node the stalest data. In freeing memory for new
data, we use a least recently used eviction policy.

```c
// Recommended max cache and object sizes.
#define MAX_CACHE_SIZE 16777216
#define MAX_OBJECT_SIZE 8388608

// Cache data structures and global variables.
typedef struct cache_node {
    char host[100], path[500];
    int size; // MAX_OBJECT_SIZE is 8388608, int is plenty
    char * data;
    struct cache_node * next, * prev;
} cache_node;

typedef struct cache_list {
    cache_node * head, * tail;
} cache_list;

cache_list cache = {NULL, NULL};
cache_list * cache_ptr = &cache;
size_t cache_size = 0;
```

### cache_info()

Given a pointer to a cache, display information about the cache.

```c
void cache_info(cache_list * cache) {
    printf("\nCACHE contents:\n");
    int total_size = 0; // MAX_CACHE_SIZE is 16777216
    cache_node * node = cache->head;
    if (node != NULL) {
        printf("- [HEAD] %s/%s (size %d)\n", node->host, node->path, node->size);
    } else {
        printf("[empty]\n");
    }
    while (node != NULL) {
        total_size += node->size;
        node = node->next;
        if (node != NULL) {
            printf("- [NEXT] %s/%s (size %d)\n", node->host, node->path, node->size);
        }
    }
    printf("\nCACHE: total cache size: %d\n", total_size);
}
```

### cache_add()

Given a pointer to a cache, add a new head node and return a pointer
to the new head node.

```c
cache_node * cache_add(cache_list * cache) {

    // Create the new head node.
    cache_node * node = (cache_node *) malloc(sizeof(cache_node));

    if (node != NULL) { // If we successfully allocated memory...

        // Deal with the new node first.
        //
        // To make this the head node of a doubly linked list, we set 
        // its `prev` pointer to NULL and its `next` pointer to the 
        // current head node of the list.
        node->next = cache->head;
        node->prev = NULL;

        // If this cache already contained at least one node, then the
        // new second node in the cache must point back to the new
        // first node in the cache.
        if (node->next != NULL) node->next->prev = node;

        // Now update the cache itself.
        //
        // If a node is inserted into an empty cache, the cache's head
        // pointer and its tail pointer both point to the same node.
        if (cache->head == NULL) cache->tail = node;

        // Update the cache's head pointer.
        cache->head = node;
    }

    return node;
}
```

### cache_remove()

Given a pointer to a cache, remove its tail node and return its data.

```c
int cache_remove(cache_list * cache) {
    int tailnode_size;

    // Get the first node in the cache.
    cache_node * curr = cache->head;

    // Handle case in which the cache contains only one node.
    if (curr->next == NULL) {
        tailnode_size = curr->size;
        #ifdef DEBUG
            printf("   removing oldest node: %s/%s (size %d)\n",
                   curr->host, curr->path, curr->size);
        #endif
        free(curr->data); // free memory of node's data
        free(curr);       // free memory of node
        cache->head = NULL;
        cache->tail = NULL;
        return tailnode_size;
    }

    // Traverse the list, preserving the previous cache node.
    cache_node * prev;
    while (curr->next != NULL) {
        prev = curr;
        curr = curr->next;
    }

    // Get the size of the data in the tail node, then free the data 
    // of the tail node and of the node itself.
    #ifdef DEBUG
        printf("Removing oldest node: %s/%s (size %d)\n",
               curr->host, curr->path, curr->size);
    #endif
    tailnode_size = curr->size;
    free(curr->data);
    free(curr);

    // Update the previous node's next pointer.
    prev->next = NULL;

    // Update the cache's tail pointer.
    cache->tail = prev;

    return tailnode_size;
} 
```

### cache_get()

Given a pointer to a cache and a URI, return a pointer to the node
containing that host and path, or NULL if the cache contains no node
with that host and path.

```c
cache_node * cache_get(cache_list * cache, char * host, char * path) {
    // Don't search without a path.
    if (strcmp(path, "") == 0) return NULL;

    cache_node * node = cache->head;
    while (node != NULL) {
        if (strcmp(host, node->host) == 0 &&
            strcmp(path, node->path) == 0) {
            return node;
        }
        node = node->next;
    }
    return NULL;
}
```

### cache_refresh()
Given a pointer to a cache and a pointer to a node, make that
node the head node of the cache.

Assumes that the specified node exists.

```c
void cache_refresh(cache_list * cache, cache_node * node) {

    // Handle case in which the specified node is already the head node.
    if (node == cache->head) return;

    // If the specified node is the tail node, make the previous node
    // the new tail node.
    if (node->next == NULL) {
        node->prev->next = NULL;

    // Otherwise, we're dealing with a middle node. Link its previous
    // and next nodes to each other.
    } else {
        node->next->prev = node->prev;
        node->prev->next = node->next;
    }

    // To make this the head node of a doubly linked list, we set 
    // its `prev` pointer to NULL and its `next` pointer to the 
    // current head node of the list.
    node->prev = NULL;
    node->next = cache->head;

    // The new second node in the cache must point back to the new
    // first node in the cache.
    node->next->prev = node;

    // Update the cache's head pointer.
    cache->head = node;
}
```



## Proxy

A sequential proxy that handles HTTP/1.0 GET requests.

```c
static const char * user_agent_hdr = "User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:10.0.3) Gecko/20120305 Firefox/10.0.3\r\n";
```

### handle_rq()

Given a client file descriptor representing a HTTP GET request,
handle the request, serving resources from the cache when possible.

Based on code of Tiny web server function `doit()`.

```c
void handle_rq(int client_fd) {

    // Data type defined in Rio (Robust I/O) package, and buffer
    // designated to hold data from the client.
    rio_t client_rio;
    char client_buf[MAXLINE];

    // GET request header components.
    char method[MAXLINE], uri[MAXLINE], version[MAXLINE];

    // Read the request line and request headers from the file
    // descriptor passed as an argument to the buffer designated to
    // hold data from the client.
    rio_readinitb(&client_rio, client_fd);
    if (!rio_readlineb(&client_rio, client_buf, MAXLINE)) return;

    #ifdef DEBUG
        printf("\n\n");
        printf("********************************************************************************\n");
        printf("                         PROXY is handling request\n");
        printf("********************************************************************************\n");
        printf("PROXY received request line:\n%s\n", client_buf);
    #endif

    // Parse the contents of the buffer holding the request line into 
    // the components: method, URI, version.
    sscanf(client_buf, "%s %s %s", method, uri, version);

    // We only accept GET requests.
    if (strcasecmp(method, "GET")) {
        fprintf(stderr, "PROXY: %s: proxy does not implement this method; ignoring\n", method);
        return;
    }

    // Parse the URI itself into hostname, port, and path.
    char host[MAXLINE], path[MAXLINE];
    char port[MAXLINE] = "80"; // set as default for HTTP
    sscanf(uri, "http://%99[^/:]:%99[^/]/%99[^\n]", host, port, path);

    #ifdef DEBUG
        printf("PROXY parsed request URI as %s\n", uri);
        printf("PROXY parsed request host as \"%s\"\n", host);
        printf("PROXY parsed request path as \"%s\"\n", path);
        printf("PROXY parsed request port as \"%s\"\n", port);
    #endif

    //
    // Search for data cached from this hostname and path. If found,
    // serve the cached data and refresh the cache to make that data's
    // node the cache's head node.
    //
    #ifdef DEBUG
        cache_info(cache_ptr);
        printf("\n");
    #endif

    cache_node * node = cache_get(cache_ptr, host, path);

    if (node != NULL) { // Cache hit
        #ifdef DEBUG
            printf("CACHE HIT on %s/%s\n\n", host, path);
        #endif
        
        // Serve data from the cache by using Rio to write to the
        // client file descriptor.
        rio_t cached_data_rio;
        rio_readinitb(&cached_data_rio, client_fd);
        rio_writen(client_fd, node->data, node->size);

        #ifdef DEBUG
            printf("PROXY: data served from cache\n");
        #endif

        // Refresh the cache.
        #ifdef DEBUG
            printf("CACHE: refreshing cache...\n");
        #endif
        cache_refresh(cache_ptr, node);

        #ifdef DEBUG
            cache_info(cache_ptr);
        #endif

        return;

    } else { // Cache miss
        #ifdef DEBUG
            printf("CACHE MISS on %s/%s\n\n", host, path);
        #endif
    }

    // Create GET request and store in a new buffer. Eventually, acting
    // as a client, we will send this request to the server.
    char request_buf[500];
    sprintf(request_buf, "GET /%s HTTP/1.0\r\n", path);

    //
    // Create request headers and concatenate with the request line.
    //
    // Hostname. Port 80 is the default for HTTP. If the port number
    // in the request line was not 80, add it.
    sprintf(request_buf, "%sHost: %s", request_buf, host);
    if (strcmp("80", port) != 0) {
        sprintf(request_buf, "%s:%s", request_buf, port);
    }
    sprintf(request_buf, "%s\r\n", request_buf);

    // Other headers.
    sprintf(request_buf, "%s%s", request_buf, user_agent_hdr);
    sprintf(request_buf, "%sConnection: close\r\n", request_buf);
    sprintf(request_buf, "%sProxy-Connection: close\r\n", request_buf);

    // Every HTTP request must be terminated with an empty line.
    sprintf(request_buf, "%s\r\n", request_buf);

    #ifdef DEBUG
        printf("PROXY: connecting to %s on port %s...\n", host, port);
    #endif

    // Open a connection to the server using a new file descriptor.
    int server_fd;
    server_fd = Open_clientfd(host, port);

    #ifdef DEBUG
        printf("PROXY: sending a GET HTTP/1.0 request...\n");
    #endif

    // Read the request from `request_buf` and send it to the server,
    // using Rio to write to the server file descriptor.
    rio_t server_rio;
    rio_readinitb(&server_rio, server_fd);
    rio_writen(server_fd, request_buf, strlen(request_buf));

    //
    // Receive the server's response, by reading it into a new buffer;
    // pass it to the client, by writing it to the client file
    // descriptor; and determine if it is cacheable.
    //

    // For writing server response to client_fd.
    char server_buf[MAXLINE];
    int response_len;

    // For parsing server response headers.
    char * content_len_str = NULL; // for Content-length header
    int content_len;                // for value of Content-length
    
    char * content_type_str = NULL; // for Content-type header
    char content_type[100];         // for value of Content-type

    int cacheable = 0;
    char cacheable_buf[MAX_OBJECT_SIZE];
    char * cacheable_buf_ptr = cacheable_buf;

    // Write the response to the client file descriptor.
    while ((response_len = rio_readnb(&server_rio, 
                                      server_buf, 
                                      sizeof(server_buf))) > 0) {
        rio_writen(client_fd, server_buf, response_len);

        // Parse Content-length and Content-type headers.
        if (!content_len_str) {
            content_len_str = strstr(server_buf, "Content-length");
            if (content_len_str) {
                sscanf(content_len_str, "%*[^:]%*c%d", &content_len);
                #ifdef DEBUG
                    printf("PROXY: parsed Content-length of response as %d\n", content_len);
                #endif
            }
        }
        if (!content_type_str) {
            content_type_str = strstr(server_buf, "Content-type");
            if (content_type_str) {
                sscanf(content_type_str, "%*[^:]%*c%s", content_type);
                #ifdef DEBUG
                    printf("PROXY: parsed Content-type of response as %s", content_type);
                #endif
            }
        }

        // Check to see if the response is cacheable, by determining
        // if it exceeds MAX_OBJECT_SIZE.
        if (!cacheable && (content_len <= MAX_OBJECT_SIZE)) {
            cacheable = 1;
            #ifdef DEBUG
                printf("\nCACHE: server's response is cacheable:\n");
                printf("       its content length (%d) is within MAX_OBJECT SIZE (%d)\n",
                       content_len, MAX_OBJECT_SIZE);
            #endif
        } else if (!cacheable) {
            #ifdef DEBUG
                printf("\nCACHE: server's response is not cacheable:\n");
                printf("       its content length (%d) exceeds MAX_OBJECT SIZE (%d)\n",
                       content_len, MAX_OBJECT_SIZE);
            #endif
        }

        // If cacheable, keep writing to buffer.
        if (cacheable) {
            memcpy(cacheable_buf_ptr, server_buf, response_len);
            cacheable_buf_ptr += response_len;
        }

    }

    // Cache the response, evicting a node if necessary.
    if (cacheable) {
        #ifdef DEBUG
            printf("CACHE: caching data...\n");
        #endif

        // Check to see if we will exceed MAX_CACHE_SIZE by caching
        // this response, and so need to evict a node. Then cache the 
        // response.
        int cacheable_size = cacheable_buf_ptr - cacheable_buf;
        if ((cache_size + cacheable_size) <= MAX_CACHE_SIZE) { // No eviction
            #ifdef DEBUG
                printf("CACHE: no eviction is necessary\n");
            #endif

            // Create new node and add its member values.
            cache_node * node = cache_add(cache_ptr);
            strcpy(node->host, host);
            strcpy(node->path, path);
            node->size = cacheable_size;
            node->data = malloc(cacheable_size);
            memcpy(node->data, cacheable_buf, cacheable_size);

            // Update size of the cache.
            cache_size += cacheable_size;

            #ifdef DEBUG
                printf("CACHE: response from %s/%s has been cached (size %d)\n",
                       node->host, node->path, node->size);
            #endif

        } else { // Evict
            #ifdef DEBUG
                printf("CACHE: evicting oldest node...\n");
            #endif
            int tailnode_size;
            tailnode_size = cache_remove(cache_ptr);
            cache_size -= tailnode_size;
        }

        #ifdef DEBUG
            cache_info(cache_ptr);
            printf("\n");
        #endif
    }

    // Close the server file descriptor.
    Close(server_fd);

    #ifdef DEBUG
        printf("\nPROXY: finished handling request\n");
    #endif
}
```

### main()

Based on `main()` in Tiny web server.

```c
int main(int argc, char ** argv) {
    printf("\n%s", user_agent_hdr);

    // Check command line arguments.
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <port>\n", argv[0]);
        exit(1);
    }

    #ifdef DEBUG
        printf("PROXY is listening on port %s\n", argv[1]);
    #endif

    // Ignore SIGPIPE signals.
    Signal(SIGPIPE, SIG_IGN);

    // Open a port to listen on as proxy.
    int listenfd;
    listenfd = Open_listenfd(argv[1]);

    // We're acting as a server. Listen for requests from clients.
    socklen_t clientlen;
    struct sockaddr_storage clientaddr;
    int connfd;
    char hostname[MAXLINE], port[MAXLINE];
    while (1) {
        clientlen = sizeof(clientaddr);
        connfd = Accept(listenfd, (SA *) &clientaddr, &clientlen);
        Getnameinfo((SA *) &clientaddr, clientlen, hostname, MAXLINE, 
                    port, MAXLINE, 0);
        handle_rq(connfd);

        // This is a sequential proxy, handling only one request at
        // a time.
        Close(connfd);
    }

    return 0;
}
```
