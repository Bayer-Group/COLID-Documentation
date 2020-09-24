# API

This section provides an overview on how to use the <u>API v3</u> of COLID.

The following use cases are described here as high-level step-by-step examples:

![UseCases](api/assets/registrationservice_use_cases.svg)

The API is available for testing and productive usage:

+ Local:  [https://localhost:51771/api](https://localhost:51771/api)
+ Docker: [https://localhost:51770/api](https://localhost:51770/api)

There is also a documentation of the whole API available:

+ Local:  [https://localhost:51771/swagger/index.html](https://localhost:51771/swagger/index.html)
+ Docker: [https://localhost:51770/swagger/index.html](https://localhost:51770/swagger/index.html)

Before the API can be used by another application, the application must be registered <u>separately</u> in Azure AD.

Please be aware that there might be further versions of the API whereby older versions will become deprecated. Information about the supported but also deprecated versions will be returned by the API in the response headers.

In order to use the API you should be familiar with the concepts of linked data and semantic web.
If this is not the case [Wikipedia (Linked Data)](https://en.wikipedia.org/wiki/Linked_data)/[Wikipedia (Semantic Web)](https://en.wikipedia.org/wiki/Semantic_Web) and the [W3C (Linked Data)](https://www.w3.org/wiki/LinkedData)/[W3C (Semantic Web)](https://www.w3.org/standards/semanticweb/) are good places to get an overview about it.
