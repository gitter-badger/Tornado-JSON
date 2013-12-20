**# This is by no means a mature package, so please excuse the mess while I get it together :)**

## Overview

Tornado-JSON is a small extension of Tornado with the intent providing the tools necessary to get a JSON API up and running quickly. See `demos/helloworld/helloworld.py` for a quick example. RequestHandler Guidelines provided below should tell you why things are how they are.

Some of the key features the included modules provide:
* Input and output *schema validation* by decorating RequestHandlers
* Automated *route generation* with `routes.get_routes(package)`
* *Automated Public API documentation* using schemas and provided descriptions
* Standardized output using the *[JSend](http://labs.omniti.com/labs/jsend)* specification


## Request Handler Guidelines

* Create an `apid` dict in each RequestHandler as a class-level variable, i.e.,
```
    class ExampleHandler(APIHandler):
        apid = {}
```

* For each HTTP method you implement, add a corresponding entry in `apid`. The schemas must be valid JSON schemas; [readthedocs](https://python-jsonschema.readthedocs.org/en/latest/) for an example. Here is an example for POST:
```
    apid["post"] = {
        "input_schema": ...,
        "output_schema": ...,
        "doc": ...,
}
```
`doc` is the **public** accompanying documentation that will be available on the wiki.

* Use the `io_schema` decorator on methods which will automatically validate the request body and output against the schemas in `apid[method_name]`. Additionally, `return` the data from the request handler, rather than writing it back (the decorator will take care of that).
```
    class ExampleHandler(APIHandler):
        @io_schema
        def post(self, body):
            ...
            return data
```

* Use `utils.api_assert` to fail when some the client does not meet some API pre-condition/requirement, e.g., an invalid or incomplete request is made. When using an assertion is not suitable, `raise APIError( ... )`; don't use JSend `fail` directly.
```
    class ExampleHandler(APIHandler):
        @io_schema
        def post(self, body):
            ...
            api_assert(condition, status_code, log_message=log_message)
```


## Documentation Generation

### Public API Usage Documentation

API Usage documentation is generated by the `tornado_json.api_doc_gen` module. The `api_doc_gen` method is run on startup so to generate documentation, simply run your app and the documentation will written to `API_Documentation.md`. in the current folder.
