# A Text Rewrite Done by AI
> I asked **llama 3.2** to improve [my original post](https://github.com/clojureman/notablog/blob/main/Working%20Around%20AWS%20API%20Gateway%20Limitations.md).
>
>  Not bad at all, actually. I think I must admit defeat.
>
> The machine beats me by being a better communicator, although it leaves out details of some importance

# Working Around AWS API Gateway Limitations

AWS API Gateway has two key limitations that can impact application performance:
* **Response size**: The maximum response size is 10MB, which can limit the amount of data that can be 
returned to clients.
* **Timeout**: The default timeout is 30 seconds, which means that if a request takes longer than this, it 
will timeout and return an error.

## A Work-Around Strategy (that Impacts Clients)

One common work-around strategy is for services to write large responses to an S3 object and return a signed 
URL to the object. This works well for large responses, but it's not enough to solve the timeout problem.

### Additional Measures

To address the timeout issue, additional measures can be taken:

* **Periodic retries**: The service can periodically retry reading the URL until some content is returned or 
until an extended time has elapsed.
	+ Pros: Easy to implement, works well for large responses and processing times
	+ Cons: Error handling can be challenging, and this approach doesn't follow REST patterns.

### An Alternative Strategy (Transparent to Clients)

For occasional "somewhat larger" and/or "slower" responses, a different strategy can be used, based on HTTP 
redirects. This approach involves:

1. **Happy path**: Returning the response normally.
2. **Large response**: Writing the response to an S3 object and returning a 302-redirect to a signed URL.
3. **Timeout**: Using a series of 302-redirects until a real result is ready or an appropriate status code 
can be returned.

### Pros

* Easy to implement
* Works well for large responses
* Clients don't need to handle extra logic
* REST patterns, including status codes, require no modification

### Cons

* Not suitable for arbitrarily long processing times (clients can only handle a limited number of redirects)
* The service needs to know how to construct a redirect to itself

