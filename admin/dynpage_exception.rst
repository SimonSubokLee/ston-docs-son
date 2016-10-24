.. _dype:

Appendix C: Dynamic Page Exceptions
***********************************

Dynamic pages are web pages developed using web programming languages (e.g. JSP, PHP), and they can change based on client requests. If the following types of dynamic pages are cached for appropriate lengths of time, it will create an exceptional decrease in the load for web servers, WAS, and databases.

- Real-time orderings (Trending search terms or rankings)
- Search results
- API for inquiries
- Detailed product pages (stock)

Pages that cannot be cached are as follows.

- **Private accounts**
   Caching a page means that if a person reads a page, other people can also read that page. Nobody wants their private information exposed to other people. From a technical standpoint, most web sites will not allow you to view private information without first logging in. The condition that checks for login status on web servers should be configured in STON's bypasses.


- **Payment**
   There are services that allow users to pay without logging in. All pages that are used for payment should be configured under caching exceptions.

- **Cookie creation**
   Cookies often contain information essential to the client. If a cookie created by User A is given to User B, a huge disaster will occur.Because of this, STON is configured to ignore all cookie headers given by the origin server.


- **Write API**
   The Read API can be effective with even 1 second of caching, but the Write API can malfunction if it is cached and a bad result is returned.

- **Ticket problems**
   Refers to reservations made in order like for concert tickets. There are many cases where this is done using WAS or a private server.
   