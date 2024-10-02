# Working Around AWS API Gateway Limitations

> Do you want to see this text rewritten by AI? 
> A mere 6GB neural network [did a better job than me](https://github.com/clojureman/notablog/blob/main/API%20GW%20limitations%20text%20rewritten%20by%20llama%203.2.md). Oh well...

## The Limitations
Two major limitations of AWS API Gateway are
- 10MB response size
- 30s timeout

## A Work-Around Strategy (that Impacts Clients)
A common work-around strategy is for the service to write the response to an S3 object and return a signed URL to the object.

Although this works fine for large responses, it is not enough the solve the timeout problem. Additional measures are required.

One such additional measure could be "periodically retry reading the URL until there some content or until some extended time has elapsed'. 
This also works fine, but error handling is not necessarily easy, and does not follow REST patterns.

Another, probably better, measure is to create some kind of internal state for responding to the client with result or status on another API route.
 
### Pros
- Easy to implement
- Works well for very large responses and for very large processing times

### Cons
- Possible need for dealing with response states inside the service
- The client needs to handle extra logic

# Another Strategy (that is Transparent to Clients)

For the *occasional* "somewhat larger" and/or "somewhat slower" response there is another possible strategy, based on HTTP redirects. 
It goes like this:

  1. Happy path (not a big or long response): 
     - Just return the response as normally.
  2. When the response is large, but well within acceptable response time:
      -  The response is written to an S3-object
      -  The service returns a 302-redirect to a signed URL to the object
  3. When the response time is getting too near the limit:
      - A series of 302-redirects is used, until a real result is ready or an appropriate status code can be returned
      - The last redirect could be a redirect to a signed URL, just as in case (2)

To make implementation easier, the redirect URLs could carry all necessary information for keeping track of time and/or retries left, as well some id for the S3 object itself.

### Pros
- Easy to implement
- Works well for very large responses
- The client doesn't need to handle extra logic
- REST patterns, including status codes require no modification

### Cons
- Not a solution for arbitrarily long processing times, since clients can only handle a limited number of redirects - 20-50 being typical.
- The service needs to know how to construct a redirect to itself
