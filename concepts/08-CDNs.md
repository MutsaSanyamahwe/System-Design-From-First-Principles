# CDNs (Content Delivery Networks)

## 1. The Problem

Imagine your application is hosted in Johannesburg.

A user in Cape Town visits your website.

The request travels a reasonable distance, and the page loads fairly quickly.

Now imagine a user in Tokyo visits the same website.


![Alt Distance affecting page laods:](../Images/00-CDN.jfif)

Every image, video, CSS file, and JavaScript bundle must travel thousands of kilometers across the internet.

Even if your server is powerful, users far away may experience:

- Slow page loads
- Higher latency
- Buffering videos
- Poor user experience


As your application grows globally, another problem appears.

Suppose millions of users request the same image.

Without any optimization:

![Alt Without optimization](../Images/06-CDN.jfif)

Your origin server must serve the same file repeatedly.

This creates:

- High bandwidth costs
- Increased server load
- Slower response times
- Scalability challenges

Engineers needed a way to:
- Move content closer to users.
- Reduce load on the main server.
- Improve performance globally.

That problem led to CDNs.

---

## 2. What is a CDN?

A CDN (Content Delivery Network) is a geographically distributed network of servers that stores copies of content closer to users.

Instead of every request going to your main server, users are served from the nearest CDN location.

---

## 3. How a CDN Works

Suppose your application is hosted in Johannesburg.

A user in Tokyo requests:

> https://pawpal.com/images/logo.png

Without a CDN:

![Alt](../Images/01-CDN.jfif)

With a CDN:

![Alt](../Images/02-CDN.jfif)

The CDN's Tokyo edge server returns a cached copy of the image.

The request no longer needs to travel all the way to South Africa.

---

## 4. The Basic Flow

![Alt](../Images/03-CDN.jfif)

- User requests content.
- DNS routes the request to the nearest CDN edge server.
- If the content is cached, the CDN returns it immediately.
- If not, the CDN fetches it from the origin server.
- The CDN caches the content for future requests.

---

## 5. What Gets Cached?

CDNs are commonly used for:
- Images
- Videos
- CSS files
- JavaScript bundles
- Fonts
- Downloads
- Static website content

Modern CDN's can also cache certain API responses.

---

## 6. Real-World Example: Netflix

![Alt](../Images/04-CDN.jfif)

Netflix does not stream every movie from a single data center.

Instead, it places video content on CDN servers around the world.

When you watch a movie:
- Your device connects to a nearby CDN server.
- The video is streamed from that local edge location.
- Latency is reduced dramatically.

---

## 7. Benefits of a CDN

- Lower latency:       Content is physically closer to users
- Faster page loads:   Reduced network travel time
- Reduced origin load: Fewer requests reach your main server
- Better scalability:  Traffic is distributed across many servers
- Higher availability: CDN's can continue serving cached content even if the origin has issues

---

## 8. CDN vs Your Application Server

A common misunderstanding is
> The CDN replaces my backend.

It doesn't.

Think of it this way:

![Alt](../Images/05-CDN.jfif)

The CDN is optimized for delivering content efficiently.

Your backend still handles:
- Authentication
- Business logic
- Database operations
- User-specific data

---

## 9. Why CDN's Matter in System Design

When users are distributed across multiple countries, serving everything from one location becomes inefficient.

CDN's solve this by:

- Reducing latency
- Reducing bandwidth usage
- Improving scalability
- Improving availability

That's why companies like:
- Netflix
- YouTube
- Amazon
- Facebook
- Cloudflare customers

all rely heavily on CDNs.

---

## 10. Key Takeaways

- A CDN is a distributed network of edge servers.
- It stores content closer to users.
- DNS usually directs users to the nearest CDN location.
- CDNs reduce latency and origin server load.
- They are primarily used for static content, though modern CDNs can cache some dynamic content as well.
- CDNs complement your backend; they do not replace it.

---
Next Step

Now that users can reach content quickly, the next question is:

"How do we control and route traffic once it reaches our infrastructure?"

 Next: Reverse Proxy
