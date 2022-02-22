Think more about authentication, so that there can be application-aware proxies
that send request on behalf of multiple users.

    Cookie: login-token=urn:uuid:aef34a443b...; ...

And in the response body (for use by programmatic clients):

    {
        "login token": URI("urn:uuid"),
        "user": URI("user")
    }
