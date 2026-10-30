1. What is the difference between PUT and PATCH?

PUT sends the whole record with all of its fields

PATCH sends only the fields that changed



2\. What does 201 tell a caller that 200 does not?
200 OK Request successful. The server has responded that your request has worked

201 Created, A new resource was created successfully



3\. Why does a program calling this API need to check for the 404?

A program checks the status code first because the object might not exist otherwise it might return a successful request that is actually an empty reply creating a false positive success



4\. Name two things a production integration needs that these requests did not have

negative testing, a staging environment

