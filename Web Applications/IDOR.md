#Web 

# Insecure Direct Object Reference

It is a type of access control vulnerability that occurs when the server receives user-supplied input to retrieve objects (files, data, documents), too much trust has been placed on the input data, and it is not validated on the server-side to confirm the requested object belongs to the user requesting it.

As an example, if visiting visiting a profile page generates a link similar to this: `http://online-service.com/profile?user_id=1000`, the `id` parameter may possibly be exploited by changing its value.

These ID parameters can also be in encoded/hashed forms, where the value has to be decrypted first and re-encoded/hashed to be passed for testing.

Other than the address bar, these parameters can exist in the content that the browser loads or the request (APIs/AJAX) that occur from interactions, both in the background. Monitoring network requests and page content may provide a hint on where this vulnerability can exist.

> Dummy/fake accounts can be used in case of dealing with unpredictable IDs.

