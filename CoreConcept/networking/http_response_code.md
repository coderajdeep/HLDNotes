HTTP response status codes tell the client **what happened to its request**. They are grouped into five classes:

| Range | Category | Meaning |
|---|---|---|
| **1xx** | Informational | Request received, processing continues |
| **2xx** | Success | Request was successfully processed |
| **3xx** | Redirection | Client needs to take another action / use another location |
| **4xx** | Client Error | Problem with the request from the client |
| **5xx** | Server Error | Server failed to fulfill a valid request |

## 1xx — Informational

These indicate that the server has received the request and is continuing to process it.

| Code | Meaning |
|---|---|
| **100 Continue** | Client can continue sending the request body. |
| **101 Switching Protocols** | Server agrees to switch to another protocol. Commonly associated with WebSocket upgrades. |
| **103 Early Hints** | Server provides preliminary information, often to allow the browser to start loading resources early. |

**1xx codes are relatively uncommon in normal application development.**

---

## 2xx — Success

These indicate that the request was successfully processed.

| Code | Meaning | Common use |
|---|---|---|
| **200 OK** | Request succeeded. | GET, PUT, etc. |
| **201 Created** | A new resource was successfully created. | POST |
| **202 Accepted** | Request accepted for processing, but processing isn't finished yet. | Async/background jobs |
| **204 No Content** | Request succeeded but there is no response body. | DELETE, updates |

### Example

```http
POST /users
```

If a user is successfully created:

```http
201 Created
```

---

## 3xx — Redirection

These tell the client that it needs to use another URL or that the requested resource has changed.

| Code | Meaning |
|---|---|
| **301 Moved Permanently** | Resource has permanently moved to another URL. |
| **302 Found** | Resource temporarily available at another URL. |
| **303 See Other** | Client should retrieve the resource from another URL using GET. |
| **304 Not Modified** | Cached version can be used; server doesn't need to send the resource again. |
| **307 Temporary Redirect** | Temporary redirect while preserving the HTTP method. |
| **308 Permanent Redirect** | Permanent redirect while preserving the HTTP method. |

For example:

```text
Browser
   |
   | GET /old-page
   v
Server
   |
   | 301 → /new-page
   v
Browser → GET /new-page
```

---

# 4xx — Client Errors

These generally mean **the request itself has a problem**.

| Code | Meaning | Typical reason |
|---|---|---|
| **400 Bad Request** | Request is invalid/malformed. | Invalid JSON, invalid parameters |
| **401 Unauthorized** | Authentication is required or failed. | Missing/invalid token |
| **403 Forbidden** | Server understood the request but refuses it. | Insufficient permission |
| **404 Not Found** | Requested resource doesn't exist. | Wrong URL/resource ID |
| **405 Method Not Allowed** | HTTP method isn't supported for that resource. | `POST /users/123` when only GET is allowed |
| **406 Not Acceptable** | Server can't provide a representation matching the client's requested format. | `Accept` header cannot be satisfied |
| **408 Request Timeout** | Server timed out waiting for the request. | Client took too long |
| **409 Conflict** | Request conflicts with the current state of the resource. | Duplicate resource, version conflict |
| **410 Gone** | Resource permanently no longer exists. | Deleted resource |
| **413 Content Too Large** | Request body is too large. | Huge file upload |
| **415 Unsupported Media Type** | Server doesn't support the request's content type. | Sending XML when only JSON is accepted |
| **429 Too Many Requests** | Client exceeded rate limit. | Too many API calls |

### Important distinction: 401 vs 403

```text
401 → "Who are you?"
403 → "I know who you are, but you aren't allowed."
```

---

# 5xx — Server Errors

These mean the **server side failed while processing the request**.

| Code | Meaning | Typical scenario |
|---|---|---|
| **500 Internal Server Error** | Unexpected server-side error. | Unhandled exception |
| **501 Not Implemented** | Server doesn't support the requested functionality. | Unsupported feature |
| **502 Bad Gateway** | Gateway/proxy received an invalid response from an upstream server. | Load balancer → backend gets bad response |
| **503 Service Unavailable** | Server is temporarily unable to handle requests. | Overload, maintenance |
| **504 Gateway Timeout** | Gateway/proxy didn't receive a response from upstream in time. | Backend is too slow/down |
| **505 HTTP Version Not Supported** | Server doesn't support the HTTP version used. | Unsupported HTTP version |

### 500 vs 502 vs 503 vs 504

This distinction is particularly important in **HLD/system design**:

```text
Client
   |
   v
Load Balancer / API Gateway
   |
   v
Backend Service
```

**500**
→ Backend itself encountered an unexpected error.

**502**
→ Gateway received a **bad response** from the backend.

**503**
→ Service is **currently unavailable**.

**504**
→ Gateway **waited too long** for the backend.

---

## The most important codes to remember

For backend/HLD interviews, I would prioritize:

```text
200 → OK
201 → Created
204 → No Content

301 → Permanent Redirect
302 → Temporary Redirect
304 → Not Modified

400 → Bad Request
401 → Authentication required/failed
403 → Forbidden
404 → Not Found
405 → Method Not Allowed
409 → Conflict
429 → Too Many Requests

500 → Internal Server Error
502 → Bad Gateway
503 → Service Unavailable
504 → Gateway Timeout
```

A simple mental model:

**2xx → "It worked."**  
**3xx → "Go somewhere else / use your cached copy."**  
**4xx → "Your request has a problem."**  
**5xx → "I (the server) had a problem processing it."**
