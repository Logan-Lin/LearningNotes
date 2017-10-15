# Attributes are not parameters!

If you are new to servlerts, you might need to spend some time reinforcing the difference between _attributes_ and _paramters_. Rest assured that when we created the exam we spent just that little bit of extra time trying to make sure we made attribute and parameter questions as confusing as possible.

<table>
    <tr>
        <td> </td>
        <td>Attributes</td>
        <td>Parameters</td>
    </tr>
    <tr>
        <td>Types</td>
        <td>Application/context<br>Request<br>Session</td>
        <td>Application/context init parameters<br>Request parameters<br>Servlet init parameters</td>
    </tr>
    <tr>
        <td>Method to set</td>
        <td>setAttribute(String name, Object value)</td>
        <td>You CANNOT set Application and Servlet init parameters -- they're set in the DD, remember?</td>
    </tr>
    <tr>
        <td>Return type</td>
        <td>Object</td>
        <td>String</td>
    </tr>
    <tr>
        <td>Method to get</td>
        <td>getAttribute(String name)</td>
        <td>getInitParameter(String name)</td>
    </tr>
</table>

# Synchronizing the service method won't protect a context attribute!

Synchronizing the service method means that only one thread in a servlet can be running at a time... but it doesn't stop _other_ servlets or JSPs from accessing the attribute!

Synchronizing the service method wounld stop other threads from the servlet from accessing the context attributes, but it won't do anything to stop a conpletely _different_ servlet.

## We don't need a lock on the serlvet... we need the lock on the context!

The typical way to protect the context attribute is to synchronize ON the context object itself. If everyone accessing the context has to first get the lock on the context object, then you're guranteed that on;y one thread at a time can be getting or setting the context attribute. But... there's still an _if_ there. If only works _if all of the other code that manipulates the same context attributes **also** synchronizes on the `ServletContext`_. If code doesn't ask for the lock, then that code is still free to hit the context attributes. But if you're designing the web app, then _you_ can decide to make everyone ask for the lock before accessing the attributes.

    public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException, ServletException {
        response.setContextType("text/html");
        PrintWriter out = response.getWriter();

        out.println("test context attributes<br>");

        synchronized(getServletContext()) {
            getServletContext().setAttribute("foo", "22");
            getServletContext.setAttribute("bar", "42");

            out.println(getServletContext().getAttribute("foo"));
            out.println(getServletContext().getAttribute("bar"));
        }
    }

## Protect session attributes by synchronizing on the HttpSession

Look at the technique we used to protect the _context_ attributes. What did we do?

You can do the same thing with session attributes, by synchronizing on the HttpSession object!

    public void doGet(HttpServlet request, HttpServletResponse response) throws IOException, ServlertException {
        response.setContentType("text/html");
        PrintWriter out = response.getWriter();

        out.println("test session attribute<br>");
        HttpSession session = request.getSession();

        synchronized(session) {
            session.setAttribute("foo", "22");
            session.setAttribute("bar", "42");

            out.println(session.getAttribute("foo"));
            out.println(session.getAttribute("bar"));
        }
    }

> ### There are no Dumb Questions
> - Isn't this overkill? Is this _really_ a possibility... that a client will open another browser window?
> - Of couse it is. Surely you've done this yourself without a second thought -- opened a second window brcause you were thired of waiting for the other one to response, or because you minimized one, or misplaced the window without realizing it, etc. The point is, you can't take the chance if you need thread-safety for your session variables. You have to know that it's quite possible for a session-scoped attribute to be used by more than one thread at a time.

# `SingleThreadModel` is designed to protect instance variables

Here's what the servlet specification says about the `SingleThreadModel` interface:

> Ensures that servlets handle only one request at a time.
> 
> This interface has no methods. If a servlet implements this interface, you are guaranteed that no two threads will execute concurrently in the servlet's service method. The servlet container can make this gurantee by synchronizing access to a single instance of the servlet, or by maintaining a pool of servlet instances and dispatching each new request to a free servlet.

## But how does the web container guarantee a servlet gets only one request at a time?

The web container vendor has a choice. The container can maintain a single servlet, but queue every request and process one request completely before allowing the next request to proceed. Or the container can create a pool of servlet instances and process each request concurrently, one per servlet instance.

# Request attributes and Request dispatching

Request attributes make sense when you want some other component of the app to take over all or part of the request. Our typical, simple example is an MVC app that starts with a servlet controller, but ends with a JSP view. The controller communicates with the model, and gets back data that the view needs in order to build the response. There's no reason to put the data in a context or session attribute, since it applies only to this request, so we put it in the request scope.

So how do we make another part of the component take over the reuquest? With a _`RequestDispatcher`_.

## `RequestDispatcher` revealed

`RequestDispatchers` have only two methods -- `forward()` and `include()`. Both take the request and response objects (which the component you're forwarding to will need to finish the job). Of the two methods, `forward()` is by far the most popular. It's very unlikely you'll use hte include method from a controller servlet; however, behind the scenes the include method is being used by JSPs in the `<jsp:include>` standard action. You can get a `RequestDispatcher` in two ways: from the request or from the context. Regardless of where you get it, you have to tell it the web component to which you're forwarding the request. In other words, the servlet or JSP that'll take over.

### Getting a `RequestDispatcher` from a `ServletRequest`

    RequestDispatcher view = request.getRequestDispatcher("result.jsp");

> The `getRequestDispatcher()` method in `ServletRequest` takes a String path for the resource to which you're forwarding the request. If the path starts with a forward slash("/"), the Container sees that as "starting from the root of this web app". If the path does NOT start with a forwards slash, it's considered _relative_ to the originla request. But you can't try to trick the Container into looking outside the current web app. In other words, just because you have lots of "../../../" doesn't mean it'll work if takes you _past_ the root of your current web app!

### Getting a `RequestDispatcher` from a `ServletContext`

    RequestDispatcher view = getServletContext().getRequestDispatcher("/result.jsp");

> Like the equivalent method in `ServletRequest`, this `getReqeustDispatcher()` method takes a String path for the resource to which you're forwarding the request, **except** you _cannot_ specify a path relative to the current resource (the one that receive this request). That means _you must start the path with a forward slash!_

### Calling `forward()` on a `RequestDispatcher`

    view.forward(request, response);

> Simple. The `RequestDispatcher` you got from your context or request knows the resource you're forwarding to -- the resource (servlet, JSP) you passed as the argument to `getRequestDisatcher()`. So you're saying, "Hey, `RequestDispatcher`, please forward this requesxt to the _thing_ I told you about earlier (in this case, a JSP), when I first got you. And here's the request and response, because that new thing is going to need them in order to finish handling the request."

# How can he track the client's answers?

Kim's design won't work unless he can keep track of _everything_ the client has already said during the conversation, not just the answer in the _current_ request. He needs the servlet to get the request parameters representing the client's choices, and save it somewhere. Each time the client answers a question, the advice engine uses _all_ o that client's previous answers to come up with either _another_ question to ask, or a finla recommendation.

**_What are some options?_**

### Use a stateful session enterprise javabean

Sure, he could do that. He could have his servlet become a client to a stateful session bean, and each time a request comes in he could locate that client's stateful bean. There are a lot of little issues to work out, but yes, you can certainly use a stateful session bean to store conversational state.

But that's _way_ too much overhead (overkill) for this app! Besides, Kim's hosting provider doesn't have a full J2EE server with an EJB Container. He's got Tomcat (a web Container) and that's it.

### Use a database

This would work too. His hosting provider _does_ allow access to MySQL, so he could do it. He could write the client's data to a database... but this is nearly as much of a runtime performance hit as en enterprise bean would be, possible _more_. And way more than he needs.

### Use an HttpSession

But you already know that. We can use an `HttpSession` object to hold the conversation state across multiple requests. In other words, for an entire _session_ with that client.

(Actually, Kim would still have to use an `HttpSession` even if he _did_ choose another option such as a database or session bean, because if the client is a web browser, Kim still needs to match a specific client with a specific database key or session bean ID, and as you'll see in this chapter, the `HttpSession` takes care of that identification.)

## How sessions work

1. - Diane selects "Dark" and hits the submit button.
- The Container sends the request to a new thread of the `BeerApp` servlet.
- The BeerApp thread finds the session associated with Diane, and stores her choice ("Dark") in the session as an attribute.

2. The servlet runs its business logic (including calls to the model) and returns a response... in this case another question, "What price range?"

3. - Diane considers the new question on the page, selects "Expensive" and hits the submit button.
- The Container sends the request to a new thread of the `BeerApp` servlet.
- The `BeerApp` thread finds the session associated with Diane, and stores her new choice ("Expense") in the session as an attribute.

4. The servlet runs its business logic and returns a response... in this case another question.

> Meanwhile, imagine another client goes to the beer site...

5. - Diane's session is still active, but meanwhile Terri selects "Pale" and hits the submit button.
- The Container sends Terri's request to a new thread of the `BeerApp` servlet.
- The `BeerApp` starts a new Session for Terri, and calls `setAttribute()` to store her choice ("Pale").

#### One problem... how does the Containre know who the client is?

The HTTP protocol uses stateless connections. The client browser makes a connection to the server, sends the request, gets the response, and closes the connection. In other words, the connection exists for only a single request/response.

_Because_ the connections don't persist, the Container doesn't recognize that the client making a second request is the same client from a previous request. As far as the Container's concerned, each request is from a new client.

## The client needs a unique session ID

The idea is simple: on the client's first request, the Container generates a unique session ID and gives it back to the client with the response. The client sends back the session ID with each subsequent request. The Container sees the ID, finds the matching session, and associates the session with the request.

#### How do the Client and Container exchange Session ID info?

Somehow, the Container has to get the session ID to the client as part of the response, and the client has to send back the session ID as part of the reqeust. The simplest and most common way to exchange the info is through cookies.

## The best part: the Container does virtually all the cookie work!

You do have to tell the Container that you want to create or use a session, but the Container takes care of generating the session ID, create a new Cookie objectm stuffing the session ID into the cookie, and setting the cookie as part of the response. And on subsequent requests, the Container gets the session ID from a cookie in the request, matches the session ID with an existing session, and associates that session with the current request.

#### Sending a session cookie in the response:

    HttpSession session = request.getSession();

That's it. Somewhere in your service method you ask for a session, and everything else happens automatically.

- You don't make the new `HttpSession` object yourself.
- You don't generate the unique session ID.
- You don't make the new Cookie object.
- You don't associate the session ID with the cookie.
- You don't set the Cookie into the response (under the Set-Cookie header).

**All the cookie work happends behind the scenes.**

#### Getting the session ID from the request:

    HttpSession session = request.getSession();

Look familiar? Yes, it's exactly the same method used to generate the session ID and cookie for the response!

    IF (the request includes a session ID cookie)
        find the session matching that ID
    ELSE IF (there's no session ID cookie OR there's no current session matching the session ID)
        create a new session

**All the cookie works happens behind the scenes.**

#### What if I want to know whether the session already existed or was just created?

Good question. The no-arg request method, `getSession()`, returns a session regardless of whether there's a pre-existing session. Since you always get an `HttpSession` instance back from that method, the only way to know if the session is new is to ask the session.

    public void doGet(HttpServletRequest reqeust, HttpServletResponse response) throws IOException, ServletException {
        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        out.println("test session attributes<br>");

        HttpSession session = request.getSession();

        if (session.isNew()) {
            out.println("This is a new session.")
        } else {
            out.println("Welcome back!");
        }
    }

#### What if I want only a pre-existing session?

You might have a scenario in which a servlet wants use only a previously-created session. It might not make sense for the checkout servlet, for example, to start a new session.

So there's an overloaded `getSession(boolean)` method just for the purpose. If you don't want to create a new session, call `getSession(false)`, and you'll get either null, or a pre-existing `HttpSession`.

#### You can do sessions even if the client doesn't accept cookies, but you have to do a little more work...

We don't agree that anybody with half a brain disables cookies. If fact, most browsers do have cookies enabled, and everything's wonderful. But there's no guarantee.

If your app depends on sessions, you need a differeny way for the client and Container ti exchange session ID info. Lucky for you, the Container can handle a cookie-refusing client, but it takes a little more effort from you.

If you use the session code on the previews pages -- calling `getSession()` on the request -- the Container tries to use cookies. If cookies aren't enabled, it means the client will never join the session. In other words, the session's `isNew()` method will always return true.

## URL rewriting: something to fall back on

If the client won't take cookies, you can use URL rewriting as a back-up. Assuming you do your part correctly, URL rewriting will always work -- the client won't care that it's happening and won't do anything to prevent it. Remember the goal is for the client and Container to exchange session ID info. Passing cookies back and forth is the simplest way to exchange session IDs, but if you can't put the ID in a cookie, where can you put it? URL rewriting takes the session ID that's in the cookie and sticks it right onto the end of every URL that comes in to this app.

Imagine a web page where every link has a little bit of extra info (the session ID) tacked onto the end of the URL. When the user clicks that "enhanced" link, the request goes to the Container with that extra bit on the end, and the Container simply strips off the extra part of the request URL and uses it to find the matching session.

#### URL rewriting kicks in only if cookies fail, and only if you tell the response to encode the URL

If cookies don't work, the Container falls back to URL rewriting, but only if you've done extra work of encoding all the URLs you send in the response. If you want the Container to always default to using cookies first, with URL rewriting only as a last resort, you can relax. That's exactly how it works (except for the first time, but we'll get to that in a moment). But if you don't explicitly encode your URLs, and the client won't accept cookies, you don't get to use sessions. If you do encode your URLs, the Container will first attempt to use cookies for session management, and fall back to URL rewriting if the cookie approach fails.

    public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        response.setContent("text/html");
        PrintWriter out = response.getWriter();
        HttpSession session = request.getSession();

        out.println("<html><body>");
        out.println("<a href='" + response.encodeURL("/BeerTest.do") + "'click me</a>");
        out.println("</body></html>");
    }